# Generic
prima di Java 1.5 non c'era type-check e quindi le liste potevano essere miste e contenere oggetti di tutti i tipi, provocando errori a runtime (non verificabili in compilazione)  
```
ArrayList files = new ArrayList();
// You do not know which kind of object are store inside the list
String filename = (String) files.get(0);
```
con i generics  
```
ArrayList<String> files = new ArrayList<>();
// You do not need the cast operation anymore
String filename = files.get(0);
```
il compilatore controlla il tipo dei dati a compile time avendo codice più sicuro  

## generic classes
la utilizziamo quando una parte della classe che dipende e può variare su più tipi  
```
class Pair<T, U> {
  private T first;
  private U second;
  // ...
}
```
di convenzione si usano lettere maiuscole alla fine dell'alfabeto  
la classe viene istanziata come segue  
```
Pair<Integer, String> pair = new Pair<>();
Integer first = pair.getFirst();
String second = pair.getSecond();
```

## generic methods
anche i singoli metodi possono essere generici  
```
public static <T> T getMiddle(T... a) { /* ... */ }
```
definite i tipi davanti al tipo di ritorno  
`T... a` permette di inserire un numero imprecisato di parametri poi viene tramutato in `Array`  
qui è il compilatore che vede dal tipo di ritorno e dai parametri quale sarà il tipo T (la priorità è sul tipo di ritorno)  
si possono restringere i tipi concessi per conoscere con certezza che implementi certi metodi  
```
class ArrayAlg {
  public static <T extends Comparable> T min(T[] a) {
    if (a == null || a.length == 0)
      return null;
    T smallest = a[0];
    for (int i = 1; i < a.length; i++)
      // We know for sure that compareTo is available
      if (smallest.compareTo(a[i]) > 0)
        smallest = a[i];
    return smallest;
  }
}
```
`T extends Comparable & Serializable` si può utilizzare sia sui metodi che sulle classi e posso estendere quante interfacce voglio ma al massimo una classe  

## type erasure
all'interno della JVM i generics non esistono quindi nel *bytecode* non c'è alcun riferimento ai tipi generici, generando alcuni problemi  
nella classe `Pair` (che non restringe i tipi) la JVM la trasforma in  
```
class Pair {
  private Object first;
  private Object second;
  // ...
}
```
e inserisce dei cast dove necessario  
a volte per far funzionare il codice ha bisogno di crearsi dei metodi di supporto per il polimorfismo, i **bridge methods**  
in `C++` creerebbe una classe per ogni combinazione dei tipi utilizzati, mentre `JAVA` mantiene il numero delle classi  
```
Pair<Employee, Salary> buddies = /* ... */;
Employee buddy = buddies.getFirst();
// After type erasure will become
Pair buddies = /* ... */;
Employee buddy = (Employee) buddies.getFirst(); // Returns an Object
```
### bridge methods
```
public class Node<T> {
  public T data;
  public Node(T data) { this.data = data; }
  public void setData(T data) {
  System.out.println("Node.setData");
    this.data = data;
  }
}
public class MyNode extends Node<Integer> {
  public MyNode(Integer data) { super(data); }
  @Override
  public void setData(Integer data) {
    System.out.println("MyNode.setData");
    super.setData(data);
  }
}
```
dopo la compilazione non abbiamo più i riferimenti ai parametri di tipo quindi su `Node` il metodo diventerà `public void setData(Object data)` e su `MyNode` verrà tolto l'`@Override` e viene creato un metodo che può fare l'override perchè ha la stessa firma  
```
public void setData(Object data) {
  setData((Integer) data);
}
```
anche se tra due classi c'è una gerarchia i tipi generici di quelle classi non mantengono la gerarchia  
con i tipi `raw` è possibile evitare i safe-check ma è **altamente sconsigliato**  
ma rimane l'ereditarità tra i tipi concreti per esempio `ArrayList<T>` implementa `List<T>` ma `T` deve essere lo stesso  

### wildcard types
se dobbiamo creare dei metodi che utilizzano le classi generiche, per aggirare il problema di tipizzazione stretta  
```
Pair<Manager> managerBuddies = new Pair<>(ceo, cfo);
Pair<? extends Employee> wildcardBuddies = managerBuddies; // OK
wildcardBuddies.setFirst(lowlyEmployee); // compile-time error
```
ma le wildcard bloccano la scrittura si possono accedere solo in lettura da usare come tipi di ritorno  
per scricerci devo usare una wildcard di altro tipo  
`Pair<? super Manager> ` da usare com parametri e non come tipi di ritorno  
le wildcard senza limiti `?` accettano come tipo di ritorno solo `Object` e come parametro solo `null` (e non è un tipo)
