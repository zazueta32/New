/*
    Programa:   shared_mem1.c
    Funcion:    Muestra el uso de memoria compartida (shared memory) como mecanismo de Comunicacion Entre Procesos (IPC)
                El programa shared_,mem1 es un "consumidor", que lee (consume) la informacion que escribe el "productor"
                en la memoria compartida y la muestra en pantalla
    Autores:    Dave Mathew y Richard Stone (Beginning Linux Programming, 4th Edition)
*/

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>

#include "shm_com.h"

int main()
{
   int x,y;
   int factorial=1;
   int running = 1;
   void *shared_memory = (void *)0;
   struct shared_use_st *shared_stuff;
   int shmid;
                // se define semilla para generacion de numeros aleatorios
   srand((unsigned int)getpid());

                // se crea area de memoria compartida
   shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
   if (shmid == -1) {
      fprintf(stderr, "Error en shmget\n");
      exit(EXIT_FAILURE);
   }

               // el proceso se asocia a la memoria compartida
   shared_memory = shmat(shmid, (void *)0, 0);
   if (shared_memory == (void *)-1) {
      fprintf(stderr, "Error en shmat\n");
      exit(EXIT_FAILURE);
   }
   printf("Memoria asociada en %X\n", (int)shared_memory);

                    // hacemos que el area de memoria compartida tenga la estructura de "shared_stuff"
   shared_stuff = (struct shared_use_st *) shared_memory;
                   // shared_stuff se define "vacio"
   
                  // ciclo hasta que el proceso lea del area de memoria compartida el texto "fin"
   while (running) {
      if (shared_stuff->written_by_you){
         x=shared_stuff->some_int[0];
 
            factorial=1;
            for(y=1;y<=x;y++){
            factorial=factorial *y;
       } 
         printf("El factorial de : %d es %d \n", shared_stuff->some_int[0],factorial);
         sleep( rand() % 4 );         /* espera por un tiempo aleatorio para que el proces productor espere */

         shared_stuff->written_by_you = 0;
         if (shared_stuff->some_int[0]== -1 )
         {
            running = 0;
         }
      }
   }

           // el proceso se des-asocia del area de memoria compartida
   if (shmdt(shared_memory) == -1)
      {
      fprintf(stderr, "Error en shmdt\n");
      exit(EXIT_FAILURE);
     }

   if (shmctl(shmid, IPC_RMID, 0) == -1) {
      fprintf(stderr, "Error en shmctl (IPC_RMID)\n");
      exit(EXIT_FAILURE);
   }
   exit(EXIT_SUCCESS);
}
