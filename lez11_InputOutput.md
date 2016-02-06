# stream I/O
Java fornisce una sola API per gestire le comunicazioni  
**input stream** è un modo per leggere byte  
**output stream** è la destinazione dei byte  
**writer** e **reader** leggono e scrivono caratteri (sequenze di byte)  
le sorgenti possono essere *file, URL, array di byte, ...* ma convergono tutte sulla stessa interfaccia (vantaggio enorme)  
`IOException` è un'eccezione checked madre delle eccezioni in lettura o scrittura  
il tipo per leggere bytes è in `java.io` da Java 7 c'è un package `java.nio` con nuove classi per fare input e output  
con `java.io.InputStream` (*Decorator*) il metodo `read()` legge un byte alla volta (il cast a byte si fa solo se non è -1 fine dello stream)  
ma c'è il metodo `read(N)` che legge un array di N bytes (ritorna un intero col numero di bytes che è riuscito a leggere per es se ce ne sono meno di N)  
lo stesso vale per `java.io.InputStream` che però va a scrivere  
*alla fine del loro utilizzo vanno chiusi col metodo* `close()` (non vengono cancellate dal garbage collector perchè vengono ancora usate e potrebbero corrompere altre comunicazioni) ma da java 7 implementano `autoclosable` [in un try with resources che crea in automatico un finally che chiude le risorse (solo se implementano autoclosable)]  

## codifica dei caratteri
esistono diversi tipi di encoding (*ASCII, UTF8, ...*) che interpretano le sequenze di bytes in modo differente, non esiste un modo di capirlo dai dati quindi chi scrive e legge deve sapere qual'è l'encoding (Java permette di fornire i diversi charset)  

## Reader e writer
per leggere i caratteri utiliziamo la struttura di lettura dei bytes  
`Reader` usa `read()` per restituire un carattere  
```
Reader in = new InputStreamReader(new InputStream(/*...*/, charset);
// Reads a code unit between 0 and 65536, or -1
int ch = in.read();
```
per leggere una linea uso il metodo `readLine()` di `BuffereReader`  
```
try (BufferedReader reader =
new BufferedReader(new InputStreamReader(
new InputStream(/*...*/)) {
// A null is returned when the stream is done
  String line = reader.readLine();
}
```
`OutputStreamWriter` può già tornare una stringa completa  
`PrintWriter` è un *Adapter* (classe con riferimento a `OutputStreamWriter`)  

se non vogliamo avere dati interpretati (big data ad alte prestazione e l'interpretazione costa), in questi casi (pochi) è preferibile leggere bytes  
`DataInput` e `DataOutput` leggono e scrivono bytes semplici, l'efficienza è che i bytes sono *fixed* sappiamo sempre di quanto spostarci  
`RandomAccessFile` è un file dove si può leggere o scrivere in qualsiasi postazione (accesso casuale), quindi devo saper interpretare cosa c'è scritto quindi i dati devono avere lunghezza fissa (si accede con `DataInput` e `DataOutput`), ci si può accedere in read (`r`) o read-write (`rw`)  
il metodo `seek()` ci dice in quale byte posizionarsi in lettura mentre il write si sposta  

se abbiamo 2 thread che scrivono nello stesso file bisogna sincronizzarli, serve il `FileLock` (funzione all'interno della stessa JVM), l'unlock viene fatto quando chiamo il `close()` su `FileChannel` (usare try with resources)  

## File e directory
`Path` è la nuova classe per astrazione dei file, perchè deve solo essere un percorso valido per il SO, può essere usato anche per le cartelle  
può essere assoluto o relativo  
se devo creare un nuovo `Path` uso la *companion class* `Paths`  
```
Path abs = Paths.get("/", "home", "rcardin"); //senza '/' in mezzo al percorso
Path rel = Paths.get("home", "rcardin");
Path another = Paths.get("/home/rcardin");
```
il path non deve corrispondere ad un file esistente  
con `Files` posso creare file o directory
```
//costruisce solo l'ultima cartella
Files.createDirectory(path);
//costruisce tutte le cartelle mancanti del path
Files.createDirectories(path);
//crea un file lancia eccezione se già esiste
Files.createFile(path);
```
questi metodi possono essere considerati atomici  

è possibile creare file temporanei con
```
Files.createTempFile(dir, prefix, suffix);
Files.createTempDirectory(dir, prefix)
```
è utile se chiamo `File.deleteOnExit()` o `StandardOpenOption.DELETE_ON_CLOSE` in creazione la JVM alla distruzione elimina i file temporanei  

spesso è utile camminare attraverso l'albero delle cartelle `walkFileTree()`  

## serializzazione
serializzare un oggetto vuol dire trasformarlo in una sequenza di bytes  
alcuni non ha senso serializzarli (thread, ...)  
se implemento `Serializable` tutto ciò che non va serializzato va accomagnato da `transient`  
`ObjectOutputStream` serializza mentre `ObjectInputStream` deserializza  
non c'è l'invocazione del costruttore, quindi se venivano fatte operazioni importanti verranno perse  
modificare il processo di serializzazione e deserializzazione è possibile (usare come primo metodo `defaultWriteObject` o `defaultReadObject` che sono quelli base)  
#### Versoning
se si serializza per un lungo tempo è possibile che la classe cambi, se si implementa `Serializable` viene richiesto un numero di versione che andrebbe cambiato (dal programmatore) ad ogni modifica della classe (una possibilità è mettergli un hashcode)
