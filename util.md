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
* pipe(p)的时候，p必须是个数组类型，能获取pipe的读和写端，0=读，1=写 <br>
* when use fork:pid=0, child ; pid>1, parent <br>


# 3.primes
- 一些重要的点：<br>
+ 将ls函数全部copy到find.c，其实我们就是改ls函数<br>
+ find path target<br>
+ 当你不知道怎么调试的时候，可以通过printf()来检查某个函数的输出结果是什么<br>
+ 递归必须在输出之后<br>
```
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

# 4.find
```


```

# 5.xargs
```



```
