# Trabalho realizado na Semana #7

## Tarefa 1

In order to crash the program we have to simply use the badfile that is provided by the `build_string.py`. This happens because in this file the string `"%.8x"*12 + "%n"` attempts to save the number of characters that come before %n in a variable. Since none is provided in the file, it will write in forbidden memory area.


## Tarefa 2


