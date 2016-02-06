# Collection framework
non si voleva avere la complessità della STL C++  
il framework vuole separare la struttura dall'implementazione  
ogni interfaccia è implementata in modo diverso in base alla esigenze  

## Collection
definisce solo 2 metodi (interessanti)  
```
public interface Collection<E> {
  boolean add(E element);
  Iterator<E> iterator();
  // ...
}
```
**Iterator** ha i suoi metodi  
```
public interface Iterator<E> {
  E next();
  boolean hasNext();
  void remove();
}
```  
i medodi di rimozione sono posti all'interno dell'iteratore per rimuoverlo in modo sicuro  
richiamare sempre `hasNext()` prima di un `next()`  per non generare errori  
le liste si scorrono con:  
```
Collection<String> c = /* ... */;
Iterator<String> iter = c.iterator();
while (iter.hasNext()) {
  String element = iter.next();
  // do something with element
}
```
da Java 5 si può usare il `foreach` *(ma solo se implementa Iterable<E>)*
```
for (String element : c) {
  // do something with element
}
```  

## Collezioni più evidenti
### LinkedList
sono doppiamente linkate  
posso ottenere in O(1) l'i-esimo elemento  
efficente per inserire e rimuovere elementi, poco efficiente per accedere all'i-esimo elemento  
si aggiunge ad iterator i metodi `add()`, `previous()` e `hasPrevious()` *(il nuovo add() aggiunge l'elemento prima dell'iteratore)*  

### ArrayList
molto efficiente ad accedere all'i-esimo elemento, ma poco efficiente nell'inserimento e rimozione (deve sistemare l'array ad ogni modifica)  
`Vector` è la prima versione di `ArrayList` e la differenza è che tutti i metodi sono sincronizzati (più lenti)  

### Set
se devo fare ricerca se un elemento è presente  
se non necessita l'ordine ma il fatto che ogni elemnt compaia al più na volta  
##### HashSet
per facilitare la comparazione degli elementi (per esempio per un `add()`) si usa `HashSet<E>` con `hashCode` (numero che deriva dall'oggetto)
ad ogni struttura dati che ha un hashCode vengono attaccati tutti gli oggetti con quell'hashCode  
una volta inserito un elemento nell'HashSet non si modifica l'elemento, altrimenti si sballa la mappa  
##### TreeSet
insieme ordinato su un certo ordine (sorted set)  
viene usato un red-blach tree, un pò più lento nell'inserimento e nella ricerca ma è ordinato  
serve una struttura dati che implementa `Comparable<T>`, altrimenti bisogna fornire l'implementazione dell'interfaccia `Comparator<T>` col suo unico metodo `compare(T a, T b)`  

### Map
non discende da `Collection` (è una radice) e non ha iteratori
coppie **chiave - valore**  
non ci possono essere 2 chiavi uguali  
se inserisco 2 elementi con la stessa chiave sovrascriverei il primo  
##### HashMap
usa un HashSet per gestire le chiavi

utilizza solo metodi `put()` e `get()`
per iterare sulla mappa ci sono delle viste *(non mutabili)* che non contengono una copia ma gli elementi  
```
Set<K> keySet() // Set of keys
Collection<K> values() // Collection of values
Set<Map.Entry<K, V>> entrySet() // Set of pairs (key,value)
```  
