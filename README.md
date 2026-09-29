# Manipulando *strings* em assembly do processador ARM

Nesta simulação, vamos aprender como manipular *strings* usando o assembly do processador ARM. 

## Exemplo

Observe o programa [minuscula.s](minuscula.s) a seguir, criado ler uma *string* da console, convertê-la para minúsculas e imprimir o resultado:

```asm
    .section .data
buffer: .space 128          @ buffer para leitura (128 bytes máx)

    .section .text
    .globl _start
_start:
    @ read(0, buffer, 128)
    mov r7, #3              @ syscall read (Linux ARM)
    mov r0, #0              @ fd = 0 (stdin)
    ldr r1, =buffer         @ buffer destino
    mov r2, #128            @ tamanho máx
    svc #0
    mov r4, r0              @ r4 = número de bytes lidos (preservar)

    @ Converter somente letras A-Z em minúsculas
    ldr r1, =buffer         @ ponteiro para buffer
    mov r5, r4              @ contador de bytes
loop:
    cmp r5, #0
    beq done                @ se não há mais bytes -> fim
    ldrb r2, [r1], #1       @ lê próximo byte e incrementa ponteiro
    cmp r2, #'A'            @ 0x41
    blt skip                @ se < 'A' -> ignora
    cmp r2, #'Z'            @ 0x5A
    bgt skip                @ se > 'Z' -> ignora
    orr r2, r2, #0x20       @ força bit 5 -> minúsculo
    strb r2, [r1, #-1]      @ grava de volta, no endereço anterior
skip:
    subs r5, r5, #1         @ decrementa contador
    b loop
done:
    @ write(1, buffer, nbytes)
    mov r7, #4              @ syscall write (Linux ARM)
    mov r0, #1              @ fd = 1 (stdout)
    ldr r1, =buffer         @ endereço do buffer
    mov r2, r4              @ número de bytes lidos
    svc #0

    @ exit(0)
    mov r7, #1              @ syscall exit (Linux ARM)
    mov r0, #0
    svc #0
```

- Na seção de dados, um único *buffer* de 128 bytes é reservado (linhas 1 e 2). 
- O *buffer* é lido usando a `syscall read` e o tamanho usado é salvo em `r4` (linhas de 8 até 13).
- Antes de entrar no laço principal, `r1` recebe o endereço do *buffer* e `r5` o número de bytes (linhas 16 e 17). No final do laço `r5` é  decrementado (linhas 29).
- O critério de parada é feito com `beq` (linhas 19 e 20).
- Apenas as letras maiúsculas e sem acento são consideradas, então os limites são testados (linhas de 22 a 25). Números e caracteres especiais são ignorados por este filtro. 
- O bit 5 do caracter lido é setado com `orr`, pois as minúsculas estão exatamente 32 posições adiante na tabela ASCII, e o byte é então gravado de volta na memória (linhas 27) usando o endereço anterior, pois o ponteiro já foi incrementado. 
- Após a conclusão do laço, as syscalls `write` e `exit` são chamadas para imprimir o resultado e finalizar o programa respectivamente. 

# Agora é a sua vez! 

1. Usando este programa como exemplo, crie um programa em [maiuscula.s](maiuscula.s) para fazer o contrário, ou seja, converter para maiúsculas. 
1. Crie também um programa em [inverte.s](inverte.s) para inverter a *string* sem modificar os caracteres. 
