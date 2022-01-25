# Appunti di Programmazione Real Time

Con il presente si vogliono raggruppare tutti i principali costrutti e funzioni Realt Time sia UNIX che RT-POSIX



## UNIX

### Time handling and Timers
In questa sezione presentiamo tutti gli elementi fondamentali per una corretta gestione del Tempo e dei Timers partendo dalle **struct** di libreria fondamentali nella definizione delle funzioni.

- **Timeval**: struct di libreria per il Tempo    
   ```c
    struct timeval{
      time_t tv_sec;      //secondi
      time_t tv_usec;     //microsecondi (1e+6)
    }
    ```
- **Gettimeofday**: funzione per ricavare il tempo trascorso dall'ultima Epoca, tempo attuale
    
    ```c
    int gettimeofday(struct timeval *tv, struct timezone *tz) //Timezone la impostiamo NULL
    ```
    
- **Type of Timer**: i Timer in unix possono essere connessi su 3 Clock differenti:
  - `ITIMER_REAL`: il real-time system clock, l'ora come la pensiamo noi
  - `ITEMER_VIRTUAL`: il tempo viartuale trascorso, ovvero il tempo che il processo ha trascorso in esecuzione
  - `ITIMER_PROF`: il tempo virtuale più il tempo che il kernel ha impiegato a schedularlo

- **Itimerval**: struct di libreria per il Timer
    ```c
    struct itimerval{
      struct timeval it_interval;    //il periodo in caso di timer periodici
      struct timeval it_value;       //tempo alla prossima attivazione 
    }
    ```
- **Setitimer**: funzione per settare il Timer che al fire signal con un SIGALARM
    ```c
    int setitimer(int witch, const struct itimerval *new_value, struct itimerval *old_value)    //parametro witch scelgo il timpo di Timer, e.g. ITIMER_REAL
    ```
- **Sleep**: funzioni per dormire n tempo
```c
#include <unistd.h>
unsigned int sleep(unsigned int seconds);
int usleep(useconds_t usec);
```

## RT-POSIX

### Time handling and Timers
In questa sezione presentiamo tutti gli elementi fondamentali per una corretta gestione del Tempo e dei Timers partendo dalle **struct** di libreria fondamentali nella definizione delle funzioni.<br>
Per poter utilizzare correttamente le funzioni proposte sarà necessario includere la Libreria RT-POSIX <time.h>


- **Timespec**: struct di libreria per il Tempo<br>
   equivalente al Timeval di UNIX con la differenza che timespec utilizza i nano secondi quindi (1e+9) di precisione, da considerare nell'utilizzo di clock e timers
   ```c
    struct timespec{
      time_t tv_sec;      //secondi
      long tv_nsec;     //nanoosecondi (1e+9)
    }
    ```
    
  Non sono presenti funzioni di libreria di Utility come potrebbe essere la somma di un tempo D al tempo preso in esame, ma possono essere semplicemente implementate dall'utente, ne vediamo un esempio
  
  ```c
  #define NSEC_PER_SEC 1000000000ULL
  void timespec_add_us(struct timespec *t, uint64_t d) {
      d *= 1000;
      t->tv_nsec += d;
      t->tv_sec += t->tv_nsec / NSEC_PER_SEC;
      t->tv_nsec %= NSEC_PER_SEC;
  }
  ```
  in tale funzione aggiungiamo dei macrosecondi, unità che spesso considereremo di base, ad un tempo espresso in secondi e nanosecondi ragion per cui dobbiamo prima connvertire al nostro attuale riferimento e poi sommare
  
- **Gettin/Setting**: funzione per ricavare e settare il tempo
   funzione che sostituisce la **gettimeofday()** di UNIX, oltre a prendere il tempo possiamo anche settarlo    
    ```c
    int clock_gettime(clockid_t clock_id, struct timespec *tp);
    int clock_settime(clockid_t clock_id, const struct timespec *tp);
    ```
   I parametri di tali funzioni sono il *clock_id* che può essere:
   -  `CLOCK_REALTIME` rappresenta il clock di sistema real time, tale valore può essere modificato con la *clock_settime()* 
   -  `CLOCK_MONOTONIC` rappresenta il clock di sistema da quando esso è partito, non può essere modificato
   -  `CLOCK_THREAD_CPUTIME_ID` rappresenta il tempo di esecuzione del thread specificato a patto che sia stato definito **_POSIX_THREAD_CPUTIME**

- **Timers** 
I processi possono creare ed inizializzare diversi timers, ognuno di essi quando fire genera un segnal event
```c
int timer_create(clockid_t c_id, struct sigevent *e, timer_t *t_id)
```
   - c_id specifica il tipo (CLOCK_REALTIME/CLCOK_MONOTONIC)
   - e la notifica dell'evento
   - t_id l'ID che verrà assegnato al timers se correttamente creato

Una volta creato il timer lo possiamo Armare con la funzione
```c
int timer_settime(timer_t timerid, int flags, const struct itimerspec *v, struct itimerspec *ov)      //approfondire itimerspec
```
   - flags se settato a TIMER_ABSTIME setta il timer al valore assoluto di v altrimenti relativo rispetto alla chiamata

- **Sleep**: funzioni per dormire n tempo
```c
#include <time.h>
int nanosleep(const struct timespec *req, struct timespec *rem);
```
a differenza del caso UNIX abbiamo una sola sleep che accetta due parametri: in *rem* verrà conservato il tempo che doveva ancora passare nel caso la sleep venga interrotta prima 
 




