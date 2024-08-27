# Regex email spiegata

Per i pigri ecco a voi la regex:

```
^(?=.{6,320}$)([\w]+([\.\-\!\+]?[\w]+)*)@((((2(([0-4]\d)|(5[0-5])))|(1\d{2})|(\d{1,2}))(\.((2(([0-4]\d)|(5[0-5])))|(1\d{2})|(\d{1,2}))){3})|([a-f\d]{4}(:[a-f\d]{4}){7})|(([\w]+([\.\-]?[\w]+)*)\.[\w]{2,63}))$
```

Per i meno pigri, vediamo di spiegarla, scomponendo la regex in parti.

## Spiegazione

La regex inizia con ^ e finisce con $ quindi effettua la validazione sull'intera email, non cerca una email all'interno di una stringa quindi esclude dal controllo:

- frasi che contengono email
- email "sporche" con caratteri prima o dopo (esempio spaziature o tabulazioni)

### lookahead sulla lunghezza

La prima parte è `(?=.{6,320}$)` che è un loockahead sulla lunghezza dell'intera stringa che dallo standard email non deve superare i 320 caratteri e per come la regex è strutturata non può essere inferiore a 6

### Parte locale

La parte locale (quella prima della chiocciola) è validata da `([\w]+([\.\-\!\+]?[\w]+)*)@` che accetta che questo pezzo della email inizia e finisca con un carattere alfanumerico (lettere, numeri e underscores) e che vi sia almeno un carattere. Inoltre accetta i simboli (. - ! +) dello standard [RFC 5322](https://datatracker.ietf.org/doc/html/rfc5322) ma mai due di seguito. La validazione della prima parte termina alla prima occorrenza della `@`.

### Dominio

Il dominio (la parte dopo la chiocciola) può rientrare in una delle seguenti casistiche:

- Indirizzo IPv4 (da 0.0.0.0 a 255.255.255.255)
- Indirizzo IPv6 (da 0000:0000:0000:0000:0000:0000:0000:0000 a ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff)
- Dominio testuale (esempio gmail.com)

Per poter analizzare i casi separatamente sono stati usati dei gruppi di cattura in OR realizzati secondo lo schema `((IPv4)|(IPv6)|(dominio testuale))`. Ne risulta la regex:

```
((((2(([0-4]\d)|(5[0-5])))|(1\d{2})|(\d{1,2}))(\.((2(([0-4]\d)|(5[0-5])))|(1\d{2})|(\d{1,2}))){3})|([a-f\d]{4}(:[a-f\d]{4}){7})|(([\w]+([\.\-]?[\w]+)*)\.[\w]{2,63}))
```

Esaminiamo i casi separatamente dal più semplice al più complesso:

#### Dominio testuale

Il dominio testuale viene validato dalla regex `(([\w]+([\.\-]?[\w]+)*)\.[\w]{2,63})` che può essere suddivisa a sua volta in `([\w]+([\.\-]?[\w]+)*)\.` (parte del dominio prima del punto) e `[\w]{2,63}` (parte del dominio dopo il punto).

La prima parte consente qualsiasi carattere alfanumerico e i siboli `.` e `-`, questi ultimi mai due volte consecutive e mai come caratteri iniziali o finali.

La seconda parte invece consente solo caratteri alfanumerici per un minimo di 2 occorrenze e un massimo di 63.

#### Indirizzo IPv6

L'indirizzo IPv6 è validato dalla regex `([a-f\d]{4}(:[a-f\d]{4}){7})` che si assicura che vi siano 8 occorrenze di un numero esadecimale a 4 cifre separati da `:` esempio `0000:abc1:fe2d:ffff:1234:0000:abc1:fe2d`

#### Indirizzo IPv4

Per l'IPv4 la regex deve validare 4 gruppi di numeri che possono assumere un qualsiasi valore compreso tra 0 e 255 e separati da punti. La regex che si occupa di questo è la seguente:

```
(((2(([0-4]\d)|(5[0-5])))|(1\d{2})|(\d{1,2}))(\.((2(([0-4]\d)|(5[0-5])))|(1\d{2})|(\d{1,2}))){3})
```

Vediamo di scomporla per analizzarla.
La regex è struttura così: `(numero da 0-255)(\.(numero da 0 a 255)){3 occorrenze}`. Questo schema dovrebbe chiarire la struttura, tuttavia vediamo nel dettaglio come è stato realizzato il gruppo di cattura per il numero da 0 a 255.

La regex è `((2(([0-4]\d)|(5[0-5])))|(1\d{2})|(\d{1,2}))`, possiamo notare anche qui una struttra in OR:
`((numero tra 200 e 255)|(numero da 100 a 199)|(numero da 0 a 99))`.

Partendo dal più semplice analizziamo la regex:

- Numero da 0 a 99 è realizzato semplicemente con `\d{1,2}` ovvero 2 cifre numeriche
- Numero da 100 a 199 è anche questo abbastanza semplice: 1 seguito da due cifre numeriche, `1[\d]{2}`
- Numero da 200 a 255 è realizzato in modo più strutturato e segue la struttura: `(2(da 00 a 49)|(5(da 0 a 5)))`, ovvero 2 seguito o da un numero tra 00 e 49 o da un numero da 50 a 55.

## Info sull'autore

- **Nome**: Urselli Cosimo
- **Email**: urselli.workmail@gmail.com
- **Hobbies**:
  - Cucina
  - Videogame
  - Romanzi fantasy/gialli
- **Occupazione**: Programmatore
