# Trabalho realizado na Semana #4

## Task 1: Manipular Variáveis de Ambiente
Varáveis de ambiente são valores atribuidos dinamicamente e usados pelo sistema operativo e outros programas. Alterações nestas variáveis, têm impacto no comportamento de muitos processos.

Em Linux, tirando partido da bash, é possível executar comandos que permitem consultar, editar e remover estas variáveis.

![Exemplo de manipulação de Variáveis de Ambiente](/Images/LB4-T1.png)

## Task 2: Transferência de Variáveis de Ambiente do Parent Process para o Child Process
Nesta task, foi usado um programa que recorre à função fork() para executar dois processos. Começou-se por compilar e correr o programa em que o child process executava a função printenv(). Em seguida, alterou-se o código para que fosse o parent a executar a função. Em ambos os casos, foi possível ver as variáveis de ambiente de cada um sendo que, guardando os outputs para dois ficheiros 
e usando o comando _diff_, foi possível verificar que eram exatamente iguais.

Foi assim possível concluir que o child process herda as variáveis de ambiente do parent process.

Nota: Após o estudo da variável global _environ_ para a task 3, foi verificado que, quando o child process é criado pelo fork(), este herda um cópia do _environ_ do parent.  

## Task 3: Variáveis de Ambiente e execve()
A função:
```
int execve(const char *pathname, char *const argv[], char *const envp[]);
```
executa o programa referido em _pathname_ com a particularidade de substituir o programa que está a ser corrido, criando uma nova stack, heap e data segments.

Ao executar um programa com a linha de código:
```
execve("/usr/bin/env", argv, NULL);
```
a função não produz qualquer tipo de output. Concluí-se então de que o programa não contém variáveis de ambiente uma vez que não as herdou do programa que substituiu.

Embora não sejam herdadas automaticamente, o argumento _envp_ permite que as variáveis de ambiente do programa que chamou a função sejam passadas para a nova execução. Para tal pode ser usado
```
extern char **environ;
```
que é um apontador para um array apontadores para strings conhecidas como o "environment". Está disponibilizado como uma variável global do _Glibc source file_.

## Task 4
O programa printa as variáveis de ambiente, uma vez que elas são herdadas. 
![Task4](/Images/LB4-T4.png)

## Task 5
* As variáveis de ambiente foram herdadas (sendo o LD_LIBRARY_PATH uma exçeção) e o programa as imprimiu
* O programa não herdou a variavel LD_LIBRARY_PATH, sendo removido antes de o SET-UID executar. Isto é uma consequencia direta da atualização introduzida no Ubuntu 9.04.
"Since Ubuntu 9.04 Jaunty Jackalope, LD_LIBRARY_PATH cannot be set in $HOME/.profile, /etc/profile, nor /etc/environment files. You must use /etc/ld.so.conf.d/*.conf configuration files. See Launchpad bug #366728 for more information." (https://help.ubuntu.com/community/EnvironmentVariables#A.2BAC8-etc.2BAC8-environment Acedido 19/11/2021)
![Task5](/Images/LB4-T5.png)


## Task 6
Nesta task foi simulado um programa malicioso que foi designado por "malicious.c". Este programa foi compilado, propositadamente dado o nome de ls e posto no diretório "/home/seed/" que é o primeiro diretório da variável "PATH".

Quando o programa "task6" foi executado, como o programa "/bin/sh" vai um a um procurar programas na "PATH" com o nome de ls, este vai encontrar o nosso "/home/seed/ls" em vez do "/bin/ls" (uma vez que o diretório "/home/seed/" aparece primeiro na variável de ambiente PATH).

O programa ls é executado com privilégios root dado que o "/bin/bash" (devido a ser executado pelo processo "task6" que sofreu elevação de privilégios) também é executado com root.

Este exploit é possível porque as variáveis de ambiente são herdadas do processo que corre o "task6" (que neste caso era um processo com a variável "PATH" propositadamente alterada).
```
// malicious.c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
    printf("Malicious code example\n");
    if (geteuid() == 0)
        printf("Executed as root\n");
    return 0;
}
```

![Task6](/Images/LB4-T6.png)
