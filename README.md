# Appunti di Programmazione Real Time

Con il presente si vogliono raggruppare tutti i principali costrutti e funzioni Realt Time sia UNIX che RT-POSIX



## UNIX

- Timeval: struct di libreria per il Tempo    
   ```c
    struct timeval{
      time_t tv_sec;      //secondi
      time_t tv_usec;     //microsecondi (1e+6)
    }
    ```
- Gettimeofday: funzione per ricavare il tempo trascorso dall'ultima Epoca, tempo attuale
    
    ```
    int gettimeofday(struct timeval *tv, struct timezone *tz) /*Timezone la impostiamo NULL*/ 
    ```
    
- Type of Timer: i Timer in unix possono essere connessi su 3 Clock differenti:
  - ITIMER_REAL: il real-time system clock, l'ora come la pensiamo noi
  - ITEMER_VIRTUAL: il tempo viartuale trascorso, ovvero il tempo che il processo ha trascorso in esecuzione
  - ITIMER_PROF: il tempo virtuale più il tempo che il kernel ha impiegato a schedularlo

- Itimerval: struct di libreria per il Timer
    ```
    struct itimerval{
      struct timeval it_interval;    //il periodo in caso di timer periodici
      struct timeval it_value;       //tempo alla prossima attivazione 
    }
    ```
- Setitimer: funzione per settare il Timer che al fire signal con un SIGALARM
    ```
    int setitimer(int witch, const struct itimerval *new_value, struct itimerval *old_value)    //parametro witch scelgo il timpo di Timer, e.g. ITIMER_REAL
    ```
