# Trabalho realizado na Semana #4

## Tarefa 2.4

Correr o comando *system("/usr/bin/env")* dá o mesmo resultado que correr *execve("/usr/bin/env", argv, environ)*, como pode ser verificado abaixo:
#

![title](images/2_4.png)
*Output de system("/usr/bin/env")*

Isto acontece porque *system()* chama a função *execl()*, que por sua vez chama *execve()*, passando por defeito um *array* infinito para as variáveis de ambiente.

## Tarefa 2.5


No Passo 3 definimos novas variáveis de ambiente, nomeadamente TEST e LD_LIBRARY_PATH. Contudo, estas novas variáveis não aparecem na listagem quando corremos o programa Set_UID compilado no Passo 2, como podemos ver no *output* abaixo:

#
![title](images/2_5.png)
*Output do programa Set_UID  compilado no Passo 2*

O resultado  é inesperado na medida em que, apesar do processo filho herdar as variáveis de ambiente já existentes no processo pai, este não herda as novas variáveis de ambiente adicionadas anteriormente.

## Tarefa 2.6



