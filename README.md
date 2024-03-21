# PROJ1-SO--Concorr-ncia
Projeto 1 de Sistemas Operacionais - Concorrência 



#define _GNU_SOURCE

#include <stdlib.h>

#include <malloc.h>

#include <sys/types.h>

#include <sys/wait.h>

#include <signal.h>

#include <sched.h>

#include <stdio.h>

#include <semaphore.h>

// 64kB stack
#define FIBER_STACK (1024*64)

struct c {
int saldo; };

typedef struct c conta;
conta from, to;
int valor;

sem_t mutex;

// The child thread will execute this function

int transferencia(void *arg) {

sem_wait(&mutex); // Entrando na região crítica


if (from.saldo >= valor) {
    from.saldo -= valor;
    to.saldo += valor;
}

printf("Transferência concluída com sucesso! \n");
printf("Saldo de c1: %d\n", from.saldo);
printf("Saldo de c2: %d\n", to.saldo);

sem_post(&mutex); // Saindo da região crítica

return 0; }

int main() {

void *stack;
pid_t pid;
int i;
// Inicializando o semáforo
sem_init(&mutex, 0, 1);

// Allocate the stack
stack = malloc(FIBER_STACK);
if (stack == 0) {
    perror("malloc: could not allocate stack");
    exit(1);
}

// Todas as contas começam com saldo 100
from.saldo = 100;
to.saldo = 100;
printf("Transferindo 10 para a conta c2\n");
valor = 10;

for (i = 0; i < 100; i++) { // Ajuste para permitir até 100 transações simultâneas
    // Call the clone system call to create the child thread
    pid = clone(&transferencia, (char *)stack + FIBER_STACK,
                SIGCHLD | CLONE_FS | CLONE_FILES | CLONE_SIGHAND | CLONE_VM, 0);
    if (pid == -1) {
        perror("clone");
        exit(2);
    }
}

// Aguardando todas as transações serem concluídas
for (i = 0; i < 100; i++) {
    wait(NULL);
}

// Liberando o semáforo
sem_destroy(&mutex);

// Free the stack
free(stack);
printf("Transferências concluídas e memória liberada. \n");
return 0;
}
