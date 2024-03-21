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
*pipe(p)的时候，p必须是个数组类型，能获取pipe的读和写端，0=读，1=写
*when use fork:pid=0, child ; pid>1, parent
<pre style="color = red;">this is pingpong_lab</pre>
**这是加粗的_ctrl b**

# 3.primes
```
```

# 4.xargs
```
```


# 5.find