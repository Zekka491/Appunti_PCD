# RMI
il server mette a disposizione i *remote object* e i loro metodi il client è colui che li consuma  

il client deve recuperare dei riferimenti agli oggetti remoti  
il **registry** conosce tutti gli oggetti e server e client gli chiedono i riferimenti agli oggetti condivisi e invocabili da remoto  

gli oggetti passati devono essere accompagnati dalla loro definizione di classe per permetterne l'utilizzo  


## Design and implements the components
### Remote interfaces
gli oggetti devono essere invocati da macchine virtuali Java (spesso diverse)  
una remote interfaces deve estendere `java.rmi.Remote` e tutti i metodi dell'interfaccia che devono essere remoti devono dichiarare `java.rmi.RemoteException` (checked) (altrimenti non posso invocarli su altre JVM)  
gli oggetti remoti vengono trattati come riferimenti (tutti gli altri vengono copiati), una rappresentazione locale viene mandata alle altre JVM  
ogni interfaccia definisce un protocollo ma con le caratteristiche di un linguaggio ad oggetti  
devo implementare i metodi della remote interfaces ma posso averne anche altri  
i paramentri possono essere di 2 dipi:  
- remote object
- local object (passati dinamicamante)  
la differenza è che remote object viene passato per riferimento (la modifica si riflette su tutti i server e client che hanno il riferimento), i locali sono passsati per copia (le modifiche hanno effetto solo su una copia)  
i locali devono essere serializzabili (implementano l'interfaccia `Serializable`)  
la **serializzazione** trasforma un oggetto in un array di bytes, ma per poterlo fare tutti i suoi campi devono essere serializzati o marcati con `transient`(che blocca la serializzazione su tutti i campi marcati quando la classe viene serializzata [tutti i campi marcati saranno inizializzati a `NULL`])  

### Security Manager
il codice scaricato in modo dinamico viene eseguito in una sandbox e gestito dal **secutrity manger** (`java.lang.SecurityManager`)  
sia client che server devono eseguire delle policy adeguate

### Exporting Remote objects
bisogna esportare gli oggetti ner registry  
dobbiamo creare lo stub  
rmi registry (**deve partire prima si qualsiasi client o server**) è predefinito nella JVM  
con `LocalRegistry` non posso indicare l'hostname (il registry deve stare sul server) quindi `Naming` è più flessibile  

## Compiling sources
compilare in RMI necessita di diversi passsati
- creare archivi JAR
```
javac compute/Compute.java compute/Task.java
jar cvf compute.jar compute/*.class
```
- scaricare automaticamente le definizioni sui web server
- costruire le classi server
```
javac -cp c:\home\ann\public_html\classes\compute.jar engine\ComputeEngine.java
```
- costruire le classi client
- definisco le security policies


## Making classes network accessible

## starting applcation
