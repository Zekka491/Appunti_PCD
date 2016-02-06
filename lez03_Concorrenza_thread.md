# Thread  
## Introduzione
- **Multitasking**
  - abilità di avere più programmi al lavoro e far sembrare ch lavorino allo stesso tempo
  - l'unità di questo tipo di programmazione è un **processo**
    - un processo è una sua variabile
    - la comunicazione tra i processi avviene attraverso un canale comune (es. socket)
      - sono abbastanza pesanti
  - tipi:
    - cooperativo: la CPU controlla il rilascio del processo
    - Time-sliced: il SO (scheduler) assegna una parte del tempo di calcolo ad ogni processo

- **Multithreading**
  - un modello di programmazione che permette a thread multipli di esistere nello stesso contesto di un singolo processo
    - ogni processo sembra fare compiti multipli
    - i thread condividono gli stessi dati e variabili
  - ogni processo è come un thread principale
    - questo thread può creare thread secondari
    - ogni thread ha un ordine di priorità (1 .. 10) `setPriority()`  
      ogni SO gestisce le priorià in modo diverso quindi si evita di usare le priorità *(nella JVM non c'è priorità e in WIN le priorità sono 7)*
  - tipi:
    - cooperativo
    - time-sliced: il thread scheduling rilascia al thread principale che usa le utilità del SO

  creare thread porta via abbastanza tempo alla JVM (il SO virtuale)  
  il thread contiene diversi **task** (pezzi di codice)  
  Processo -> Thread -> Task

## Gestione dei thread in Java
i `Thread` in Java sono le primitive che eseguono i tasks  
l'interfaccia `Runnable` descrive un task da far partire, normalmente concorrentemente agli altri  
il codice di un metodo run sarà eseguito in un thread
```
public interface Runnable {
  void run();
}
```
i tasks hanno vita breve per qusto non si deve perdere tempo a far partire un thread  
se invoco il medodo start() su un thread creo un nuovo thread dove andrò ad eseguire nuovi task
```
Runnable task = () -> {
  int i = 0;
  while (i < 1000)
    i++;
}
new Thread(task).start(); // A thread running a task
```
bisogna disaccoppiare i task che vanno in parallelo dal meccanismo di esecuzione  
*Estendere la classe Thread non è più raccomandato*, se si hanno molti task è più dispendioso creare un singolo thread per ognuno
```
class MyThread extends Thread {
  public void run() {
    // task code, don’t do this!!!
  }
}
```
**non si invoca mai il metodo run() di un Thread o di una istanza Runnable altrimenti non si fa multithreading**, il task è semplicemente eseguito nello steso thread  

il `main` esegue nel **main thread** dal quale si possono eseguire altri thread  
i thread possono essere di due tipi:
  - **User threads**: la JVM si ferma quando non ci sono più user thread in esecuzione
  - **Deaon threads**: un deamon si ferma quando si ferma il suo user thread  
  per definirlo `setDeamon()`

il metodo `run()` non può lanciare eccezioni unchecked

### Ciclo di vita dei thread
per conoscere lo stato si usa `getState`
1. New: se appena stato crato con il new(), non ancora eseguito
2. Runnable: appena viene invocato il metodo start() (è diverso da running)
3. Blocked e Waiting:
  + quando accede ad un oggetto in lock
  + quando cerca id eseguire qualcosa che è già eseguito da altri
  + chiamata di `Thread.sleep(t)`
  + sta aspettando che un altro thread finisca (`join()`)
5. Time Waiting:
6. Terminated:
  + viene eseguita una return
  + se viene terminata l'ultima operazione
  + se si presenta un'eccezione non gestita

è possibile inviare una richiesta di interruzione (non si può più bloccare un altro thread dall'esterno) usanto il metodo `interrupt`

- Join: aspettare un altro thread con `join()` o `join(t)`  
- Sleep: il thread viene messo a dormire e dopo il tempo t viene di nuovo riportato a runnable, se però il thread su cui viene chiamato `sleep` era in stato interrupted si genera un `InterruptedException` e lo stato viene portato a `false`
- Yield: serve per abbandonare volontariamente l'esecuzione di un thread **ma non si usa quasi mai**
