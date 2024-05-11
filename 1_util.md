# 1.sleep
```
int main(int argc, char *argv[])  
{
  if(argc < 2){
    fprintf(2, "Usage: sleep [time]\n");
    exit(1);
  }

  int time = atoi(argv[1]);
  sleep(time);
  exit(0);
}
```
# 2.pingpong
### 注意：
* pipe(p)的时候，p必须是个有两个空间的数组，能获取pipe的读和写端，0=读，1=写 <br>
* when use fork:pid=0, child ; pid>1, parent <br>
* parent need to use wait(), child need to use exit().
```
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]){
      int p2c[2], c2p[2];
      if(pipe(p2c) < 0){
            printf("pipe panic\n");
            exit(-1);
      }
      if(pipe(c2p) < 0){
            printf("pipe panic\n");
            exit(-1);
      }
      printf("p2c[0]: %d,p2c[1]: %d\n",p2c[0],p2c[1]);
      printf("parent_read_fd:%d, parent_write_fd:%d\n",c2p[0],p2c[1]);
      printf("kid_read_fd:%d, kid_write_fd:%d\n",p2c[0],c2p[1]);
      int pid = fork();
      if(pid == 0){
            //this is child process
            char buf[10];
            read(p2c[0],buf,10);
            printf("%d:kid has received ping\n", getpid());
            write(c2p[1],"p",1);
      }else if (pid > 0)
      {     
            //this is parent process
            char buf[10];
            write(p2c[1],"c",1);
            read(c2p[0],buf,10);
            printf("%d:parent has received pong\n",getpid());
      }
      close(p2c[0]);
      close(p2c[1]);
      close(c2p[0]);
      close(c2p[1]);
      exit(0);
}
```

# 3.primes

### 思路：
+ 递归调用fork(),每个子进程输出第一个数，并且筛选掉第一个数的整数倍<br>
+ 要记得每个进程关掉不必要的fd<br>

```
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

void seive(int left_pipe[2]){//筛子
      //读取left_pipe的第一个数
      int p;
      read(left_pipe[0], &p, sizeof(int));
      if(p == -1){
            //是哨兵，到最后了
            exit(0);
      }

      //按要求输出
      printf("prime: %d\n", p);

      //关闭不用的管道：左管道的写_fd
      close(left_pipe[1]);

      //创建右管道
      int right_pipe[2];
      pipe(right_pipe);
      
      int pid_n = fork();
      if(pid_n != 0){//当前进程
            //关闭不用的管道：右边的管道的 读_fd
            close(right_pipe[0]);

            //筛选，将不是第一个数的倍数的数写给右管道
            int buf;
            while (read(left_pipe[0], &buf, sizeof(buf)) != -1)
            {
                  if(buf % p != 0){
                        write(right_pipe[1], &buf, sizeof(buf));
                  }
            }

            //补写哨兵
            buf = -1;
            write(right_pipe[1], &buf, sizeof(buf));

            //最后又关掉这些管道
            close(left_pipe[0]);
            close(right_pipe[1]);

            wait(0);
      }else if(pid_n == 0){//下个进程
            seive(right_pipe);
            exit(0);
      }
}

int main(int argc, char *argv[]){
      //创建第一个pipe
      int left_pipe[2];
      pipe(left_pipe);

      int pid = fork();
      if(pid == 0){
            seive(left_pipe);
            exit(0);
      }else if(pid > 0){
            //向第一个pipe里写入2-32
            for (int i = 2; i < 33; i++)
            {     
                  write(left_pipe[1], &i , sizeof(i));
            }
            close(left_pipe[0]);
            close(left_pipe[1]);
      }
      wait(0);
      exit(0);     
}

```

# 4.find
### 一些重要的点:
+ 将ls函数全部copy到find.c，其实我们就是改ls函数<br>
+ find path target<br>
+ 当你不知道怎么调试的时候，可以通过printf()来检查某个函数的输出结果是什么<br>
+ 递归必须在输出之后<br>
```
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

//格式化name，即将./console->console
char*
fmtname(char *path)
{
  static char buf[DIRSIZ+1];
  char *p;

  // Find first character after last slash.
  for(p=path+strlen(path); p >= path && *p != '/'; p--)
    ;
  p++;

  // Return blank-padded name.
  if(strlen(p) >= DIRSIZ)
    return p;
  memmove(buf, p, strlen(p));
  memset(buf+strlen(p), ' ', DIRSIZ-strlen(p));
  return buf;
}
  
//判断是否能够递归，能就返回1，如果是. or .. 就返回0
int
norecurse(char *path){
      char *buf = fmtname(path);
      if(buf[0] == '.' && buf[1] == ' ')
            return 0;
      if(buf[0] == '.' && buf[1] == '.' && buf[2] == ' ')
            return 0;
      return 1;
}

void
find(char *path, char *target)
{ 
  char buf[512], *p;
  int fd;
  struct dirent de;
  struct stat st;

  if((fd = open(path, 0)) < 0){
    fprintf(2, "find: cannot open %s\n", path);
    return;
  }

  if(fstat(fd, &st) < 0){
    fprintf(2, "find: cannot stat %s\n", path);
    close(fd);
    return;
  }

  //判断strcmp()函数返回的数是什么---32,所以和后来查找到的条件是等于32
  // int a = strcmp(fmtname(path), target);
  // printf("%d \n ", a);

  //测试fmtname()对原来的path干了什么
  //printf("path: %s, fmtname(path): %s\n", path, fmtname(path));

  switch(st.type){
  case T_FILE:
  //根据把这两行注释掉和不把这两行注释掉的调试，发现他们两没用，因为文件里不会有文件
     //printf("%s %d %d %l\n", fmtname(path), st.type, st.ino, st.size);
     //printf("path: %s   fmtname(path): %s\n", path, fmtname(path));
    break;

  case T_DIR:
    if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
      printf("find: path too long\n");
      break;
    }
    strcpy(buf, path);
    
    //look buf
    //printf("buf is :'%s'\n", buf);
    
    p = buf+strlen(buf);
    
    //look p 通过两个look可以得出为啥递归
    //printf("p is :'%s'\n", p);
    
    *p++ = '/';
    while(read(fd, &de, sizeof(de)) == sizeof(de)){
      if(de.inum == 0)
        continue;
      memmove(p, de.name, DIRSIZ);
      p[DIRSIZ] = 0;
      if(stat(buf, &st) < 0){
        printf("find: cannot stat %s\n", buf);
        continue;
      }

      //查找，如果匹配就输出path
      if(strcmp(fmtname(buf),target) == 32){
      printf("%s\n", buf);
      }

      //如果能递归就递归
      if(norecurse(buf)){
            find(buf,target);
      }
    }
    break;
  }
  close(fd);
}

int
main(int argc, char *argv[])
{

  if(argc < 2){
    printf("usage: find [path] [target]\n");
    exit(0);
  }

  if(argc == 2){
      find(".", argv[1]);
  }

  if(argc == 3){
      find(argv[1], argv[2]);
  }
  exit(0);
}

```

# 5.xargs
### steps:
+ 要求是echo bye | xargs echo hello too 输出hello too bye
+ 首先fd 0 能够用来读取管道的标准化输入
+ 其次用字符串数组接受xargs的参数
+ 最后是参数的拼接，若从fd 0读出的输入出现"\n",将其改为\0并拼接前半部分的参数，用数组的for循环实现
+ 通过exec(command, arguments)来实现xargs后接的第一个command

```
#include "kernel/types.h"
#include "kernel/param.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

#define MAXSIZE 16
int main(int argc, char *argv[]){
      //echo xsq | xargs echo nice to meet you


      //读取标准化输入--stdin--fd=0
      //stdin = xsq
      char stdin[MAXSIZE];
      read(0, stdin, MAXSIZE);

      //获取自己的参数
      //xargv = echo[0] nice[1] to[2] meet[3] you[4] 
      char *xargv[MAXARG];
      int xargc = 0;
      for(int i = 1; i < argc; i++){
            xargv[xargc] = argv[i];
            xargc++;  
      }
      

      char *p = stdin;
      for(int j = 0; j < MAXSIZE; j++){
            if(stdin[j] == '\n'){//如果在stdin里出现"\n",将其变为0，并拼接到xargv
                  int pid;
                  pid = fork();
                  if(pid == 0){
                        //child
                        stdin[j] = 0;
                        xargv[xargc] = p;
                        xargc++;
                        xargv[xargc] = 0; 
                        
                        exec(xargv[0], xargv);
                        exit(0);
                  }else if(pid > 0){
                        //parent
                        p = &stdin[j+1];
                        wait(0);
                  }
            }
      }
      wait(0);
      exit(0);
}


```
