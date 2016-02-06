# Socket
processi sono distribuiti se agiscono su ambiti divisi (non condividono variabili)  
un processo esegue su un **host** (nodo della rete internet)  
la comunicazione arriva almeno al livello TCP  
ma macchine diverse implementano i metodo di comunicazione dei processi in modo diverso, ma dobbiamo poter parlare allo stesso modo (Java lo fa)  
un'applicazione client-server è una comunicazione point-to-point su un canale della rete esterna  
per permettere la comunicazione abbiamo bisogno dei `socket`  
il server attende sulla sua porta l'arrivo del socket dal client (per identificare il server deve conoscere host e porta)  
ogni volta che una comunicazione client-server viene accettata il server cambia indirizzo TCP
- il server ascolta a TCP(localhost,777)
- il client si connette TCP(hostname,post,chostname,port)

ogni connessione TCP è unica  
in Java esiste `java.net.Socket` (client) e `java.net.ServerSocket` (server) e servono a mascherare l'architettura dei socket del SO  
comunicare via socket è un sistema di basso livello, ridefinendo il protocollo di comunicazione di handshake

`Socket` implementa `AutoCloseable` che mi garantisce la chiusura nel giusto ordine delle connessioni usate in qualsiasi caso  
quando apro il socket lato client il socket lato server deve già essere aperto  

bisogna definire il protocollo (definizione dei messaggi che si possono scambiare client e server)  


la parte server è passiva, attende un messaggio dal client  
la creazione di un `ServerSocket` richiede solo la porta  
quando viene contattato si crea un socket e si comporta esattamente come la parte client  
per accettare più client basterebbe mettere la condizione dentro un while e quando crea un socket lo mette dentro un thread o runnable  
