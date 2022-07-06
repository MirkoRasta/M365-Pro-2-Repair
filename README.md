# M365-Pro-2-Repair
Guida e promemoria di tutte le riparazioni che ho dovuto fare sul monopattino xiaomi Pro 2


Partiamo dal misfatto: il precedente proprietario urta contro uno scalino lateralmente alla ruota anteriore, proprio dove ci sono i fili del motore, tranciandoli e facendoli toccare insieme. Dopo questo evento il monopattino era completamente morto e decide quindi di venderlo per 30€.

Lo acquisto e comincio la ricerca del guasto tramite uno schema che ho trovato in rete.

Breve principio di funzionamento:
La batteria alimenta costantemente la scheda che però viene tenuta spenta. Il positivo della batteria va all'emettitore del transistor Q2, che si accende mettendo la base a massa quando si preme il tasto sul display (Connettore a 4 PIN). Quando si preme il tasto viene portato alto il terminale di Enable del regolatore switching marchiato BDX (in basso a sinistra nello schema, nel riquadro verde) che porta la tensione dai circa 40V della batteria a 12V. I 12V sono utilizzati da U3, un'altro regolatore switching per produrre la 5V necessaria al display e a tutto il resto della scheda, tranne il processore, che viene alimentato a 3,3V ricavati dalla 5V tramite U5, un regolatore lineare classico. Una volta accesi tutti questi regolatori restano funzionanti finche non si stacca la batteria o il processore tramite il pin 4 porta il segnale di enable sotto la soglia che spegne il regolatore a 12V, spengendo tutta la scheda.

L'ipotesi di guasto è che la tensione di una delle 3 fasi del motore (quindi la tensione della batteria) sia entrata in contatto con la 5V che alimenta i sensori a effetto hall presenti all'interno del motore che servono per ricavarne la posizione.

In effetti anche accendendo il regolatore a 12V (che chiameremo U1 da qui in avanti) la 12V è presente ma mancano la 5V e di conseguenza anche la 3,3V.

Misurando invece la resistenza in parallelo ai condensatori di alimentazione del processore (Ad esempio C33 e C34), leggevo un valore molto basso, indice di un possibile  cortocircuito nel regolatore a 3,3V o addirittura nel processore.
Una volta rimossi sia U3 che U5 il corto è rimasto, segno della rottura del processore che una volta rimosso ha restituito l'alta impedenza alla linea.

Al posto di U3 ho messo provvisoriamente uno di quei modulini step down regolabili di aliexpress, mentre al posto di U5 un AMS1117 3.3 saldato con dei fili sulle piazzole dell'integrato rimosso.
Per il processore fortunatamente in passato avevo comprato un CANusb (che non ero mai riuscito ad usare con il suo programma in cinese) che incredibilmente montava lo stesso processore del monopattino (a meno della lettera "B" o "8" dell'STM32F103CBT6 che non riuscivo a distinguere sul vecchio processore del motopattino) ed è stato quindi felice di donare il suo organo principale per lo scopo.

Collegato all'STLINK quando ancora era saldato sul CANusb ci scrivo sopra tramite il programma m365prorec e procedo alla sostituzione.

Collego la scheda al monopattino con i minimi collegamenti possibili e processo all'accensione, con quasi successo, visto che il monopattino è vivo ma il display non mostra nulla e produce prima il segnale di errore 18. Questo errore era dovuto alla mancata alimentazione dei sensori di Hall del motore, i 5V mancavano perchè si era rotto il diodo marchiato T4 (MBR0520) posto in serie alla 5V(Questo diodo non è presente nello schema che ho trovato ma è in serie al primo pin del connettore a 5 PIN). Poi l'errore 29 perché non avevo collegato le fasi del motore per precauzione, poi il 14, che dovrebbe essere un errore dovuto all'acceleratore. A questo punto quindi procedo a smontare il display per testare l'acceleratore che però funzionava, restituendo una tensione di 0.870V al minimo e circa 4V al massimo. C'era però qualcosa che non andava nel display visto che il modulo bluetooth diventava incandescente dopo pochi secondi dall'accensione. Nella scheda era presente un regolatore identico a U5 per alimentare il modulo Bluetooth con la 3,3V, che però era in corto circuito e gli mandava direttamente la 5V, falsando i riferimenti di lettura dell'acceleratore(che viene letto tramite un paritore che divide per circa 4). Ripetuto lo stesso lavoro della scheda del motore l'errore è sparito e il monopattino ha iniziato a funzionare, seppur con errore 27 continuo. 
L'errore 27 chiamato "Bad controller password" era dovuto al codice casuale che avevo messo nel programma di recupero della centralina motore, non avendo la targhetta del vero numero di serie presente sul mio monopattino. Una volta riflashato il firmware della scheda motore con un codice plausibile (in europa 21886/numeri a caso) l'errore è sparito e il monopattino funzionava regolarmente senza problemi.

Quasi.

Visto che il display non voleva in nessun modo funzionare e non riuscivo a spiegare al vecchio proprietario come rimuovere il monopattino dalla sua applicazione così che potessi collegarmici io ho deciso di insistere su questo punto. Per l'accoppiamento col bluetooth è bastato flashare un firmware a caso sul BLE tramite BLEM365ProRec e mi ha permesso di collegarmi ma ancora senza display. Verificata la presenza dell'alimentazione all'integrato TM1637 (un controller per display a 7 segmenti degli orologi) e i segnali di dati e clock dell'SPI usato nella comunicazione fra modulo BT e integrato ho deciso di acquistarlo, prendendo direttamente su ebay un modulo completo di display per un orologio che costava meno dell'integrato singolo. Sostituito l'integrato il display finalmente ha iniziato a funzionare.

Quasi.

Perché adesso si vede tutto ma la velocità resta su 0 e l'indicatore della batteria lampeggia di rosso nonostante la batteria sia al 65% dall'applicazione.
Per fortuna la soluzione stavolta è stata rapida, è stato sufficiente flashare un firmware sulla scheda motore trame xiamoflasher e finalmente adesso TUTTO funziona alla perfezione.

Fine del patimento, 33,5€ per un M365 PRO 2, ci si pole stare.


