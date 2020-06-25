<h1> The library redpwnCTF writeup</h1>

<p> First we donwload the files provided to us which consists of a c source file a binary, and a libc file which is just a copy of the libc used on the server</p>
<p> As the description says, there are not a lot of useful Functions in the binary itself, so we will have to find them elsewhere</p>
<p> First we can take a look at the .c source file and discover there is a buffer overflow vulnerability </p>

```c

int main(void)
{
  char name[16];

  setbuf(stdout, NULL);
  setbuf(stdin, NULL);
  setbuf(stderr, NULL);

  puts("Welcome to the library... What's your name?");

  read(0, name, 0x100);
  puts("Hello there: ");
  puts(name);
}

```
<p> read() is able to read more bytes then fit into name buffer therefore its vulnerable to a buffer overflow</p>
<p> Instead of returning to code inside the binary we will be returning to code inside libc library making this a ret2libc attack. To pass system()
 its argument we will need to be in control of rdi register. A search for gadgets that can accomplish this will find one that is useful to us.</p>
 <p>$ ropper -- file the-library --search "% ?di"</p>
 
 0x0000000000400733: pop rdi; ret;
 
 <p> We can use this to pass system its argument and to leak addresses to find system() functions address. Because ASLR is enabled, we cant 
  simply take the address and use it remotely. We will have to leak it before returning to it. </p>
  
  <p> By leaking the puts address from libc we can determine the libc version and find commonly known offsets which will also help us 
  calculate the libc base and calculate the system() function address to return to.</p>
  
  <p> Once we have the system() address we can rop chain to pop_rdi gadget and pop in the address of /bin/sh string and return back to 
  system() function and get our shell </p>
  
  
 <p> Once we leak puts address from libc we can throw it into libc online database and find the offsets we need to calculate everything else.
  Below is a script i wrote whitch will connect to target server and leak puts libc address and calculate the rest of the addresses
  and use them to set rdi value to /bin/sh string and return back to system() and get shell </p>
  
<a href="../../exploit.py">Exploit.py</a>
