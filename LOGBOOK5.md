# Trabalho realizado na Semana #4

## Tarefa 3

No programa `call_shellcode.d`, é criada uma função func e é colocado no seu endereço de memória um código binário que abre uma shell. 

```c
char code[500];
strcpy(code, shellcode); // Copy the shellcode to the stack
int (*func)() = (int(*)())code;
func(); // Invoke the shellcode from the stack
```

Utilizando o ficheiro `Make` fornecido, são criados dois binários: `a32.out` e `a64.out`. Executando qualquer um destes podemos observar que é aberta uma shell. Isto acontece porque o programa chama a função `func`. No seu endereço de memória está o programa binário, sendo este o código executado.