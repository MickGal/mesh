```
# Algoritmo B.A.T.M.A.N.

L'algoritmo B.A.T.M.A.N. (Better Approach To Mobile Adhoc Networking) è un protocollo di routing efficiente, pensato per le reti ad hoc mobili (MANET) e le reti mesh. Il suo principio fondamentale è la decentralizzazione della conoscenza delle rotte, il che significa che ogni nodo non ha bisogno di avere una visione completa della topologia della rete. Invece, ogni nodo si concentra solo sui propri vicini immediati e sul miglior passo successivo per raggiungere una destinazione.

## Punti chiave dell'algoritmo B.A.T.M.A.N.

- **Conoscenza decentralizzata del percorso**: Ogni nodo mantiene informazioni solo sul prossimo miglior passo per raggiungere altri nodi, piuttosto che sull'intero percorso. Questo riduce la complessità nel mantenere una topologia completa della rete e consente alla rete di adattarsi facilmente ai cambiamenti.

- **Flooding basato su eventi**: B.A.T.M.A.N. utilizza un meccanismo di flooding basato su eventi per la scoperta delle rotte. Questo approccio aiuta a limitare il numero di messaggi di controllo, riducendo così il sovraccarico e prevenendo informazioni topologiche contraddittorie e obsolete, che potrebbero causare loop di routing.

- **Operazione senza tempo**: L'algoritmo non si basa su aggiornamenti programmati o time-out per ottimizzare le decisioni di routing, il che lo rende più resiliente ai cambiamenti della rete. Invece, reagisce agli eventi, come guasti ai collegamenti o l'arrivo di nuovi nodi, per adeguare dinamicamente le decisioni di instradamento.

- **Gestione dei collegamenti non affidabili**: B.A.T.M.A.N. è progettato per funzionare in modo efficiente in ambienti con collegamenti instabili o che cambiano frequentemente, poiché non richiede che ogni nodo mantenga una visione stabile dell'intera topologia della rete.

Questo rende l'algoritmo particolarmente adatto per reti dinamiche e dense, dove mantenere una consistenza globale della topologia può essere sia costoso che impraticabile.

## Descrizione dell'algoritmo

L'algoritmo del protocollo B.A.T.M.A.N. può essere descritto (in modo semplificato) nel seguente modo: ogni nodo trasmette messaggi broadcast (chiamati messaggi di origine o OGMs) per informare i nodi vicini della propria esistenza. Questi vicini rilanciano i messaggi OGMs secondo regole specifiche, per informare i propri vicini dell'esistenza dell'iniziatore originale di questi messaggi, e così via. In questo modo, la rete viene inondata di messaggi di origine. 

Gli OGMs sono di piccole dimensioni, con una dimensione tipica di pacchetto grezzo di 52 byte, compreso l'overhead IP e UDP. Gli OGMs contengono almeno l'indirizzo dell'origine, l'indirizzo del nodo che trasmette il pacchetto, un TTL (Time-To-Live) e un numero di sequenza.

Gli OGMs che seguono un percorso con una qualità del collegamento wireless scarsa o saturata subiscono una perdita di pacchetti o ritardi nel loro cammino attraverso la rete mesh. Pertanto, gli OGMs che viaggiano su percorsi migliori si propagano più velocemente e in modo più affidabile.

Per determinare se un OGM è stato ricevuto una o più volte, esso contiene un numero di sequenza, assegnato dall'origine del messaggio. Ogni nodo rilancia ogni OGM ricevuto al massimo una volta e solo quelli ricevuti dal vicino identificato come il miglior passo successivo (best ranking neighbor) verso l'iniziatore originale dell'OGM.

In questo modo, gli OGMs vengono inondati selettivamente attraverso la rete mesh e informano i nodi riceventi dell'esistenza di altri nodi. Un nodo X impara dell'esistenza di un nodo Y ricevendo i suoi OGMs, quando questi vengono rilanciati dai vicini a singolo hop. Se il nodo X ha più di un vicino, può determinare, dal numero di messaggi di origine ricevuti più rapidamente e in modo più affidabile tramite uno dei suoi vicini a singolo hop, quale vicino scegliere per inviare dati al nodo distante.

L'algoritmo seleziona quindi questo vicino come il miglior passo successivo verso l'origine del messaggio e configura di conseguenza la sua tabella di routing.
```
