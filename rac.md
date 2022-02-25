## Oracle RAC

> Un cluster è un gruppo di computer indipendenti ma interconnessi che agiscono come se fossero un sistema singolo…è usato per aumentare la disponibilità e le performance. 

**Clusterware: software che provvede varie interfacce e servizi per un cluster.**

Quindi:
- permette al cluster di essere gestito come un'unica entità
- protegge l'integrità del cluster
- mantiene un registro di risorse sul cluster
- provvede una vista comune di risorse

Il clusterware fa parte della Grid Infrastructure, integrata con ASM. 
La base dell'ASM è ACFS(ASM CLUSTER FILE SYSTEM, un cluster file system che può essere usato per la maggiorparte dei dati basati su file, come docs o report)

Il clusterware gestisce le risorse come gli indirizzi VIP, DB, listeners e servizi.

TFA: Trace File Analyzer
RHP: Rapid Home Provisioning

Oracle Extended Cluster consiste in nodi che sono posizionati su più posti chiamati sites.

Il clusterware è facile da installare, facile da gestire ed è integrato al RAC, non sono richiesti ulteriori clusterware.

**La rete del Clusterware**

- Ogni nodo deve avere almeno 2 Network Adapter(uno per rete dati pubblica e uno per le private)
- Ogni network adapter pubblico deve supportare il protocollo TPC/IP
- L'adapter interconnect dovrebbe supportare :
  1. UDP/RDS
  2. TPS per piattaforme Windows per comunicazioni DB
- Le piattaforme devono usare GIPc (Grid Interprocess communication)

SCAN: indirizzo usato dai clienti per connettersi al cluster. È un hostname collocato nel sottodominio GNS registrato a 3 indirizzi IP. 
Lo SCAN provvede un nome stabile, disponibile ai clienti e indipendente dai nodi che formano il cluster.

Oracle ACFS:elemento grafico che mi dice di creare dei fs. / fs condiviso fra tutti i nodi.--> ci metti solitamente le cose di Oracle
-Multi piattaforma, file system scalabile e gestione di spazio che estende le funzionalità di Oracle ASM.
-Supporta file DB Oracle e altri file
-Spalma dati su dischi per bilanciare il carico
-Provvede mirroring integrato sui dischi
ACFS comunica con ASM ed è configurato con lo storage ASM

* RAC ha una relazione una a molti tra DB e istanza
* DB Non cluster hanno una relazione uno a uno tra DB e istanza
* Un DB RAC può avere  100 istanze

Il RAC è posizionato solitamente su un solo datacenter

Oracle DATAGUARD è consigliato nel caso di necessaria protezione dei dati in caso di corruzione, disastri naturali o ecc…

Rete interconnect è una rete privata che connette tutti i server nel cluster. La rete interconncet usa almeno un switch e un adattatore Gigabit Ethernet

SCALEUP= capacità di mantenere gli stessi livelli di performance quando entrambi i workload e le risorse aumentano proporzionalmente.

Scaleup: volume parallel/volume original

SPEEDUP= effetto che si ottiene applicando un numero incrementale di risorse a una quantità fissa di lavoro da fare per raggiungere una riduzione proporzionale in tempi di esecuzione

Speedup: time original/time parallel

I nodi devono essere sempre sincronizzati fra di loro così che ogni istanza vede la versione più recente di un blocco e i buffer cache.

## TABLESPACES 

Le tablespace possono avere tre tipi di oggetti:

	- Oggetti permanenti
	- Oggetti temporanei 
	- Segmenti undo

I segmenti UNDO salvano lo stato precedente ad una modifica del dato, che rende consistente il risultato di una query.

Redo e Undo non sono opposti: Redo protegge tutte le modifiche nel blocco, Undo è semplicemente un segmento. 

La temporary tablespace è occupata da quegli oggetti non permanenti.

Se lo spazio disponibile nella PGA (Program Global Area; memoria privata allocata alla sessione) non è sufficiente in una determinata sessione, allora si usa una tablespace temporanea per eseguire operazioni che richiedono spazio temporaneo. Queste operazioni sono le join tables, la creazione di indici…ogni utente può avere una temporary tablespace. 

SYSRAC è il privilegio amministrativo di default per entrare in RAC, serve per evitare di usare SYSDBA (che è abbastanza rischioso)

```
Per connettersi come SYSRAC:
CONNECT / AS SYSRAC
CONNECT / @db1 as SYSRAC
CONNECT / @db2 as SYSRAC
```

In un DB RAC, ogni istanza deve avere almeno due gruppi di  file redolog.
Quando creiamo un db con DBCA, esso posiziona i redolog nelle istanze. Si può modificare il numero di redolog e la loro dimensione, sia durante che dopo la creazione del db. 

Quando il redolog in uso si riempie, l'istanza comincia a scrivere sul prossimo file. 
Se il db è in ARCHIVEMODE, ogni istanza deve salvare i log compilati come dei log archiviati che sono tracciati nel controlfile. 

Durante il recovery del db, tutte le istanze abilitate sono controllate per vedere se il recovery è necessario. 

Per i db gestiti da amministratori, ogni istanza ha il suo redolog. 

Per aggiungere un redo log in un'istanza, nello statement 
ALTER DATABASE ADD LOGFILE

Aggiungere la clausola 
INSTANCE

Se non si specifica l'istanza dove aggiungere il redo, allora il redo verrà inserito nell'istanza attualmente connessa. 

V$LOG è la vista dove si possono vedere tutti i redolog con il thread a cui appartengono
V$THREAD è la vista per vedere il thread attualmente utilizzato

## UNDO

Il db oracle gestisce in maniera autonoma i segmenti undo senza una specifica undo tbs che è assegnata ad una specifica istanza.
Tutte le istanze possono leggere i blocchi di undo per potere leggere in maniera consistente il dato.
Qualsiasi istanza può aggiornare qualsiasi UNDO TABLESPACE durante la transaction recovery, però la undo tablespace in quel momento non deve essere utilizzata da altre istanze.

Per assegnare una undo tablespace in RAC bisogna specificare un valore nel parametro UNDO_TABLESPACE in ogni istanza nel spfile. 

## LOCAL TEMP TBS

Non possono essere utilizzate per archiviare oggetti come tabelle o indici.
Sono state create da Oracle per migliorare le operazioni I/O. 

Queste tbs sono usate per migliorare la gestione delle temp tablespace nelle istanze in sola lettura.

Per creare delle local temp tbs si usa l'estensione FOR LEAF | FOR ALL

Possono essere create nelle istanze READ-ONLY e READ-WRITE

Quando si crea una temp tbs locale questa risulta creata su tutte le istanze.

Un utente può avere due temp tbs di default:

	- Un local temporary per quando l'utente è connesso all'istanza read only.
	- Un temp tbs da essere usato quando l'utente è connesso alle istanze READ-WRITE.



Le viste USER_TABLESPACES e DBA_TABLESPACES hanno la colonna SHARED, che specifica se è LOCALE o CONDIVISA

Le istanze RAC possono essere accese e spente usando 
Srvctl 
SQLPlus 
Enterprise Manager 

Con srvctl:

![image](https://user-images.githubusercontent.com/96287973/155706609-0d4f1933-1348-42be-b48c-98c8f277490e.png)

Con SQLPlus 

![image](https://user-images.githubusercontent.com/96287973/155706664-370cb3ab-3a77-4fa5-b6ef-f69ec4a4aaf0.png)

> Per verificare le istanze attive si può usare v$active_instances 

Gli SPFILE sono utilizzati se si usa DBCA, devono essere creati in un diskgroup ASM. 
Se invece il db è stato creato manualmente, bisogna creare un SPFILE da un PFILE
Tutte le istanze usano lo stesso SPFILE

L'SPFILE è un file binario, non è editabile come un file qualsiasi, bisogna cambiare le impostazioni del parametro con ORACLE EM oppure con ALTER SYSTEM 

Oracle RAC usa i PFILE solo se SPFILE non esiste oppure se gli viene specificato PFILE nel comando STARTUP. 
