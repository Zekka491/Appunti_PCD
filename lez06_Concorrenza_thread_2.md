# THREAD’S ADVANCED CONCEPTS
## Callable
è un task  
rappresenta una elaborazione asincrona di tipo `Runnable` che ritorna un valore  
il valore tornato non è direttamente disponibile abbiamo bidogno di un tipo per rappresentare il valore  
```
public interface Callable<V> {
// The method can throw an exception,
unlike Runnable run method
  //
public V call() throws Exception;
}
```

## Future
il **future** rappresenta un valore disponibile solo in futuro ed è *immutabile*  
```
public interface Future<V> {
  V get() throws . . .;
  V   get(long timeout, TimeUnit unit) throws . . .;
  void cancel(boolean
  mayInterrupt);

  boolean isCancelled();
  boolean   isDone();
}
```
il metodo `get()` si blocca finchè non viene reso disponibile il risultato o scade il timeout  
`FutureTask` è un Adapter per usare un `Callable` ottenendo un `Future`
- Adapter delle interfacce `Runnable` e `Future`
- serve per eseguire un `Callable` usando un `Thread`  

```
Callable<Integer> myComputation = ...;
FutureTask<Integer> task = new FutureTask<Integer>(myComputation);
Thread t = new Thread(task); // it's a Runnable
t.start();
// . . .
Integer result = task.get(); // it's a Future
```
è immutablie e tratta la programmazione asincrona in modo sincrono

## Executors
è un'interfaccia con metodi factory  
ogni volta che chiamo un Thread devo parlare con il SO ed è molto inefficiente creare per ogni task un Thread  
meglio fare eseguire più task ad un solo Thread o fare un solo Thread per un grande task  
quindi converrebbe creare i task e lasciare ad altri l'onere di creare i thread  
`Executors` sono dei *Thread pool* una coda di Thread a disposizione  
```
// Create the thread pool with a specified execution policy
Executor executor = Executors.newCachedThreadPool();
Runnable hellos = new Runnable() { /* Say hello a lot of times */ };
Runnable goodbyes = new Runnable() {/* Say hello a lot of times */ };
// Submit task for execution to thread pool
executors.execute(hellos);
executors.execute(goodbyes);
```
decide lui a quale Thread far eseguire il task  
è basato sul pattern *producer/consumer*
- attività producono i task
- thread consumano i task
divisione dalla sottomissione all'esecuzione dei task  
il comportamento dipende dalle policy di esecuzione (semplificando molto il lavoro del client)

### Metodi factory
- **newCachedThreadPool** New thread are created as needed; idle threads are kept for 60 seconds
- **newFixedThreadPool** The pool contains a fixed set of threads; idle threads are kept indefinitely
- **newSingleThreadExecutor** A «pool» with a single thread that executes the submitted tasks sequentially (similar to the Swing event dispatch thread)
- **newScheduledThreadPool** A fixed-thread pool for scheduled execution
- **newSingleThreadScheduledExecutor** A single-thread «pool» for scheduled execution

### Service
per eseguire dei `Callable` devo utilizzare `ExecutorService`  
**Lifecycle**
- *Running* è lo stato in cui viene creato l'Executor, accetta nuovi tasks
- *Shutting down* non accetta nuovi tasks ma termina quelli già in lista
- *Terminated* quando ha terminato tutti i task in lista dallo stato shutting down

## Deadlock
ci sono metodologie per vedere se un Thread sta andando in deadlock e cercare di recuperare (la JVM non lo fa)  
