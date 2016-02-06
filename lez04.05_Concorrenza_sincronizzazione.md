# Sincronizzazione nella programmazione concorrente
ai giorni nostri molti pc hanno più core che possono essere sfruttati con i thread  
si può sempificare il lavoro dei singoli thread suddividendo i compiti *(es calcoli con interfaccia grafica)*
## Rischi dei thread
### Safety hazard
anche programmi banali in single-thread possono causare errori su thread multipli
```
public class UnsafeSequence {
  private int value;
  /** Returns a unique value. */
  public void next() {
    value++; // Three operations: read, add and store
  }
  public int getValue(){
    return value;
  }
 public static void main(String [] args){
   final UnsafeSequence us = new UnsafeSequence();
   for(int i=0;i<10000;i++){
     new Thread(new Runnable(){
       @Override
       public void run() {
         us.next();
       }
       }).start();
   }
   Thread.sleep(60000);
   System.out.println(us.getValue());
 }
}
```
### Deadlock & Livelock
**Deadlock** se non sincronizzo bene i thread uno potrebbe non partire mai perchè sta aspettando risorse da un altro e l'altro attende delle risorse che ha lui  
**Livelock** il sistema non è bloccato ma un thread si e lo resta per sempre

## Thread Safety
tutte le condizioni non sono sul timing dei thread ma si basano sullo **stato** condiviso (share) e mutabile (mutable) dei thread  
3 passi per avere un programma thread-safety:
- non ho stati condivisi tra i thread
- rendere lo stato immutabile (gli oggetti senza stato o con stato immutabile sono *sempre* thread safety)
- utilizzare la Sincronizzazione

se il mio programma non segue almeno uno dei punti sono sicuro che **NON** è thread-safety

## Race Condition
avviene quando la correttezza di un'esecuzione avviene sul timing di thread multipli  
bisogna rendere le **operazioni atomiche** (tutti i suoi passi vengono eseguiti)
- read-modify-write: legge modifica e riscrive
- check-then-act: controlle ed eseguo  

### Compound action
sono operazioni divise che però devono essere eseguite come una sola  
ci sono classi che rendono le read-modify-write e le check-then-act atomiche in automatico: `AtomicInteger, ... `  
se lo stato condiviso è maggiore di una variabile questo sistema non basta più

### Synchronized Block
#### Intrinsic lock
rende un intero blocco di operazioni atomiche  
in java c'è la keyword `synchronized`
```
public synchronized void method() {
  // method body
}
// ...is equivalent to
public void method() {
  this.intrinsicLock.lock();
  try {
    // method body
  } finally {
    this.intrinsicLock.unlock();
  }
}
```
questo metodo è atomico ma è molto inefficiente in quanto magari non tutto il codice deve essere sincronizzato

#### Reentrant lock
si chiama *reentrant* perchè un oggetto può richiamare più volte il metodo `lock()` sullo stesso oggetto `lock`, non crea errori perchè c'è un contatore nascosto che incrementa alla chiamata `lock()` e decrementa con `unlock()` il lock viene liberato quando il contatore torna a 0  
questa funzione permette di poter fare subclassing delle classi con lock  
```
public class SafeCachedSequence {
  private Lock lock = new ReentrantLock();
  private int lastValue;
  private int value;
  public int getNext() {
    // Invariant of the class is now satisfied
    lock.lock();
    try {
      lastValue = value;
      value = value + 1;
      int result = value;
    } finally {
      lock.unlock();
    }   
    return result;
  }
}
```
l'unlock è inserito in un blocco finally per essere sempre sicuri di rilasciare alla fine dell'esecuzione sia che le cose vadano bene che male  

esiste un misto tra i due tipi di lock che usa il lock implicito ma si può scegliere la parte di codice su cui eseguirlo
```
public class SafeCachedSequence {
  private Object lock = new Object();
  private int lastValue;
  private int value;
  public int getNext() {
    synchronized (lock) {
      lastValue = value;
      value = value + 1;
      int result = value;
    }
    return result;
  }
}
```

## Condition
usiamo le condizioni per evitare deadlock  
possiedo il lock ma devo soddisfare una codizione allora chiamo sull'oggetto `condition` il metodo `await` che rilascia il lock e attende la risorsa  
```
class Bank {
  private Condition sufficientFunds;
  public Bank() {
    // Getting a condition with an evocative name
    sufficientFunds = bankLock.newCondition();
  }
}
```  
`sufficientFunds.await();`  
quando un altro processo che ha usato il lock finisce chiama un `signalAll()` sulla risorsa che risveglia gli altri thread che aspettavano quella condizione e esegue quelli che possono proseguire
`sufficientFunds.signalAll();`  
sveglia tutti i thread che aspettavano la risorsa e vengono riaggiunti alla coda dei thread running e quando la CPU gli darà l'accesso partiranno (se possono)
devo ricontrollare ogni volta che la risorsa sia libera quindi di solito si mette await dentro un while che verifica la condizione
```
while (!(/* ok to proceed */)) {
  condition.await();
}
```  
quando cambiamo lo stato condiviso utilizziamo un `signalAll` per avvertire gli altri thread in await su quella condizione del cambiamento
```
public void transfer(int from, int to, int amount) {
  bankLock.lock();
  try {
    while (accounts[from] < amount)
      sufficientFunds.await();
    // transfer funds
    sufficientFunds.signalAll();
  } finally {
    bankLock.unlock();
  }
}
```  
gli **Intrinsic Lock** agiscono su tutto il metodo ma hanno sempre e solo una condizione e i metodi di attesa (`wait()`) e risveglio (`notifyAll()`) appartengono alla classe Object  
avendo una sola condizione su tutti i lock siamo meno efficienti dovendo risvegliare tutti ad ogni risveglio  

### Consigli
- i lock e synchronized non andrebbero mai usati perchè java possiede dei meccanismi più efficienti
- nel caso si debbano usare sarebbero meglio gli Intrinsic Lock
- usare i Lock/Condition se proprio dobbiamo

## Visibilità
il vero problema è la visibilità dei valori del nostro stato a tutti i thread  
è complicato dall'architettura moderna (la RAM è lenta) quandi si cerca di fare **caching** sui registri (in cache) per ottimizzare le prestazione  
**reordering** il compilatore, la JVM e il processore possono riordinare le istruzioni per velocizzare l'esecuzione, chiaramente non devono modificare la semantica del programma, ma *è stata programmata per programmi sequenziali* quindi nei programmi concorrenti può creare problemi
```
public class Reordering {
  private static boolean guard = false;

  public static void main(String [] args) {
    Thread hellower = new Thread(new Runnable() {
      @Override
      public void run(){
        for(i=0;i<10000;i++) {
          System.out.println("Hello " + i);
        }
        guard = true;
      }
    });

    Thread goodbyer = new Thread(new Runnable() {
      @Override
      public void run(){
        int i = 0;
        while(!guard){
          i++;
        }
        System.out.println("Goodbye " + i);
      }
    });
  }  
}
```
nella JVM si può limitare il reordering
- una variabile `final` è sempre visibile dopo l'iniializzazione
- una variabile `static` è sempre visibile dopo l'iniializzazione
- le modifiche ad una variabile `volatile` sono visibili
- cambiamenti successe prma di rilasciare un lock sono visibili a chiunque acquisisce il lock  
utilizzare final per il maggiorn numero di variabili risulta quindi molto consigliato

### volatile
le variabili volatili non vengono mai tenute in cache e vengono ogni volta recuperate in RAM e per questo motivo sono sempre visibili ai Thread in quanto ogni volta deve andarla a leggere non risolvono però l'atomicità  
nell'esempio di prima si poteva definire `private static volatile boolean guard = false;` per risolvere il problema  
sono consigliate per lo status flag (check)  

## Thread Confinement
la maggior parte delle classi della JDK non sono sincronizzate (per non incidere sulle prestazioni), quindi forniamo ad ogni thread una diversa istanza si un oggetto utilizzando i `ThreadLocal` (generics)  
invece di condividere lo stato viene replicato e quando i lthred termina diventa garbage
```
public class A{
  private static ThreadLocal<SimpleDateFormat> dateFormat = new ThreadLocal<SimpleDateFormat>() {
    @Override
    protected SimpleDateFormat initialValue() {
      return new SimpleDateFormat("yyyy/MM/dd");
    }};
  }

  public static void main(String [] args) {
    for(int i = 0;i<100;i++) {
      new Thread(new Runnable(){
          @Override
          public void run() {
            System.out.println(dateFormat.get().format(new Date()));
          }
        });
    }
  }
}
```
## Immutability
tanti problemi della concorrenza derivano dall'accesso condiviso a stati mutabili  
JAVA non fornisce una definizione formale dell'immutabilità  
un oggetto è immutabile se:
- il suo stato non può essere modificato dopo la sua costruzione
- tutti i propri campi sono marcati `final`
- la classe deve essere costruita correttamente (non devo passare il `this` al di fuori della classe dal costruttore)  
non è presente il `const` ma il final può essere acceduto senza sincronizzazione addizionale  
il final non permette di ridefinire l'oggetto ma si può modificare lo stato dell'oggetto (es. un array)  
nelle classi immutabili se costuiamo un quasiasi oggetto che non è primitivo dobbiamo sempre costruirlo con una copia altrimenti chi ci ha richiamato mantiene un riferimento all'oggetto  
```
/*
 * Errato dal punto di vista logico il compilatore non da errore
public OneValueCache(BigInteger i, BigInteger[] factors) {
  lastNumber = i;
  lastFactors = factors);
}
*/
public OneValueCache(BigInteger i, BigInteger[] factors) {
  lastNumber = i;
  lastFactors = Arrays.copyOf(factors, factors.length);
}
```
