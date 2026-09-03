# BlackNick

```text
┌─────────────┐
│ J           │
│ ♠           │
│             │
│     ♠       │
│    ╱│╲      │
│   ╱ │ ╲     │
│  ╱  │  ╲    │
│     J       │
│             │
│           ♠ │
│           J │
└─────────────┘
```

**BlackNick** è un gioco di Blackjack per browser realizzato come applicazione web monolitica in un singolo file HTML.

La versione documentata da questo README è **BN V1.0**.

Il progetto include interfaccia, logica di gioco, grafica, musica, effetti sonori e stato applicativo direttamente nel documento HTML. Il multiplayer utilizza **PeerJS + WebRTC** e non richiede un backend applicativo proprietario.

> BlackNick utilizza esclusivamente chips virtuali. Non gestisce denaro reale, pagamenti, account, depositi, prelievi o transazioni economiche.

---

## Indice

- [Scopo del progetto](#scopo-del-progetto)
- [Caratteristiche principali](#caratteristiche-principali)
- [Modalità di gioco](#modalità-di-gioco)
- [Regole Blackjack implementate](#regole-blackjack-implementate)
- [Regole configurabili](#regole-configurabili)
- [Puntate e chips](#puntate-e-chips)
- [Oracolo](#oracolo)
- [Statistiche](#statistiche)
- [Audio e feedback visivo](#audio-e-feedback-visivo)
- [Architettura](#architettura)
- [Multiplayer P2P](#multiplayer-p2p)
- [Randomizzazione](#randomizzazione)
- [Gestione dello shoe](#gestione-dello-shoe)
- [Sicurezza applicativa](#sicurezza-applicativa)
- [Dipendenze di rete](#dipendenze-di-rete)
- [Self-contained: cosa significa realmente](#self-contained-cosa-significa-realmente)
- [Requisiti](#requisiti)
- [Avvio](#avvio)
- [Come giocare](#come-giocare)
- [Flusso di una partita](#flusso-di-una-partita)
- [Fine del match](#fine-del-match)
- [Disconnessioni](#disconnessioni)
- [Limiti tecnici](#limiti-tecnici)
- [Troubleshooting](#troubleshooting)
- [Debug e test interni](#debug-e-test-interni)
- [Struttura del progetto](#struttura-del-progetto)
- [Compatibilità](#compatibilità)
- [Privacy e persistenza](#privacy-e-persistenza)
- [Licenza](#licenza)
- [Disclaimer](#disclaimer)

---

# Scopo del progetto

BlackNick nasce come **client Blackjack portabile**, pensato per poter essere copiato, inviato, archiviato e avviato senza dover mantenere una struttura composta da decine di file separati.

Gli obiettivi principali sono:

- offrire un Blackjack completo direttamente nel browser;
- permettere partite private tra amici senza registrazione;
- evitare un backend applicativo dedicato;
- mantenere l'interfaccia e la logica il più possibile nello stesso documento;
- avere musica, effetti sonori e asset grafici incorporati;
- supportare sia il gioco in solitaria sia il multiplayer;
- mantenere l'host come autorità della partita;
- fornire un'esperienza grafica più vicina a un piccolo gioco desktop che a una semplice pagina web.

BlackNick **non ha come scopo**:

- il gioco d'azzardo con denaro reale;
- la gestione di account;
- il matchmaking pubblico;
- una piattaforma competitiva anti-cheat;
- la persistenza cloud;
- la simulazione certificata di un casinò reale;
- la sostituzione di un server di gioco professionale.

Il progetto è quindi adatto soprattutto a:

- partite private;
- demo;
- sperimentazione WebRTC;
- sviluppo UI/UX;
- studio della logica Blackjack;
- distribuzione portabile come singolo documento HTML.

---

# Caratteristiche principali

## Gameplay

- Blackjack contro dealer automatico.
- Modalità Solo.
- Multiplayer P2P.
- Fino a **4 giocatori**.
- Hit.
- Stand.
- Double Down.
- Split.
- Double After Split.
- Surrender.
- Insurance.
- Blackjack naturale.
- Push.
- Gestione automatica del dealer.
- Dealer S17 o H17.
- Payout Blackjack 3:2 o 6:5.
- Numero di mazzi configurabile.
- Chips iniziali configurabili.
- All-In.
- Ripetizione ultima puntata.
- Eliminazione dei giocatori rimasti senza chips.
- Fine match automatica.
- Possibilità di terminare il match di comune accordo.

## Multiplayer

- Stanze tramite codice.
- Codice stanza di 6 caratteri.
- PeerJS 1.5.5.
- WebRTC DataChannel.
- Host autoritativo.
- Stato sincronizzato tra i peer.
- Handshake applicativo.
- Controllo stanza piena.
- Controllo peer duplicati.
- Timeout di connessione.
- Gestione disconnessioni.
- Tentativo di riconnessione al broker PeerJS.
- Passaggio da Solo a multiplayer quando un amico entra in un momento compatibile.

## Interfaccia

- UI dark casino / cyber-luxury.
- Lobby separata dal tavolo.
- Indicatore delle fasi.
- Dock azioni.
- Pannello regole.
- Pannello statistiche.
- Pannello cronologia.
- Pannello audio.
- Pannello Oracolo.
- Animazioni delle carte.
- Animazione saldo.
- Feedback puntata.
- Overlay dedicati per vittoria, sconfitta, pareggio e Blackjack.
- Effetti grafici associati alle winning streak.
- Cambio della scena visiva tra lobby e partita.

---

# Modalità di gioco

## Solo

L'host può iniziare una partita Solo quando è l'unico giocatore presente nella stanza.

La modalità Solo utilizza lo stesso sistema di:

- shoe;
- puntate;
- dealer;
- regole;
- statistiche;
- audio;
- risultati.

Se un secondo giocatore entra mentre è in corso una partita Solo, BlackNick può segnalarlo come multiplayer in attesa.

Il passaggio avviene in una fase compatibile, evitando di interrompere arbitrariamente una mano già in esecuzione.

Quando viene effettuato il passaggio:

- il match Solo viene chiuso;
- lo shoe viene rigenerato;
- il round torna a 0;
- i saldi vengono riportati alle chips iniziali;
- le statistiche di sessione vengono resettate;
- inizia il multiplayer.

### Nota importante

La modalità Solo **non costituisce attualmente una modalità completamente offline indipendente**.

L'interfaccia crea prima un host PeerJS e inizializza lo stato della stanza; solo successivamente permette all'host di avviare il Solo.

Di conseguenza, nella V1.0 l'avvio standard della modalità Solo dipende comunque dal caricamento di PeerJS e dall'apertura del peer.

---

## Multiplayer

Il multiplayer supporta fino a:

```text
4 giocatori
```

L'host crea una stanza e riceve un codice di 6 caratteri.

Gli altri giocatori inseriscono il codice e si collegano direttamente all'host tramite WebRTC.

Il server PeerJS viene utilizzato principalmente per il signaling necessario alla creazione della connessione.

Una volta instaurato il DataChannel, lo stato di gioco viene scambiato tra browser.

---

# Regole Blackjack implementate

## Valore delle carte

- 2–10: valore nominale.
- J, Q, K: 10.
- Asso: 11 quando possibile, altrimenti 1.

Il valore dell'Asso viene ridotto automaticamente quando la mano supererebbe 21.

---

## Blackjack naturale

Il Blackjack naturale richiede esattamente due carte:

```text
Asso + carta da valore 10
```

Sono quindi validi:

```text
A + 10
A + J
A + Q
A + K
```

Le mani generate da uno split **non ricevono il payout Blackjack naturale**.

---

## Hit

`Hit` pesca una nuova carta.

Se il totale:

- supera 21 → Bust;
- raggiunge 21 → la mano viene automaticamente chiusa;
- resta sotto 21 → il giocatore può continuare.

È presente una piccola protezione temporale contro hit ripetuti involontari.

---

## Stand

`Stand` chiude volontariamente la mano corrente.

Nel multiplayer il turno passa al giocatore successivo.

---

## Double Down

Il Double è disponibile quando:

- la mano corrente contiene esattamente 2 carte;
- è presente una puntata valida;
- il giocatore dispone di chips sufficienti a coprire un'altra puntata equivalente.

Il Double:

1. scala una seconda puntata;
2. pesca una carta;
3. chiude la mano.

### Double After Split

Il codice consente il Double anche sulle mani nate da Split quando:

- la mano split selezionata contiene 2 carte;
- il giocatore ha saldo sufficiente.

Quindi:

```text
DAS = supportato
```

---

## Split

Lo Split è consentito quando:

- la mano contiene esattamente 2 carte;
- le due carte hanno lo stesso rank;
- la mano non è già una mano split;
- il giocatore dispone delle chips necessarie per duplicare la puntata.

Esempi splittabili:

```text
8 + 8
K + K
A + A
10 + 10
```

Il controllo viene effettuato sul **rank**, non semplicemente sul valore numerico.

Per esempio:

```text
K + Q
```

non viene considerato una coppia splittabile, anche se entrambe valgono 10.

### Numero di split

L'implementazione corrente crea due mani e non supporta un ulteriore re-split delle mani già separate.

### Split degli Assi

Quando vengono splittati due Assi:

- viene assegnata una carta aggiuntiva a ciascun Asso;
- entrambe le mani vengono poi chiuse.

Quindi:

```text
Split Aces → una carta aggiuntiva per mano
```

---

## Surrender

Il Surrender è disponibile su una mano:

- non splittata;
- composta da 2 carte;
- con puntata valida.

Quando viene utilizzato:

- la mano termina;
- viene restituita metà della puntata;
- l'altra metà viene persa.

La restituzione viene gestita con precisione a una cifra decimale per le chips.

---

## Insurance

L'Insurance viene proposta quando:

- Dealer Peek è attivo;
- la carta scoperta del dealer è un Asso.

La puntata assicurativa equivale a:

```text
puntata principale / 2
```

Se il dealer ha Blackjack:

```text
profitto Insurance = 2 × stake Insurance
```

Esempio:

```text
Puntata principale: 1.000
Insurance:            500
Profitto Insurance: 1.000
```

L'Insurance richiede saldo sufficiente.

---

# Regole configurabili

Le regole vengono impostate dall'host nella lobby.

Gli altri giocatori ricevono le regole sincronizzate.

## Impostazioni disponibili

| Regola | Valori | Default |
|---|---|---:|
| Dealer Peek | ON / OFF | ON |
| Chips iniziali | 1.000–100.000 | 10.000 |
| Oracolo | ON / OFF | ON |
| Numero mazzi | 1 / 2 / 4 / 6 / 8 | 1 |
| Soft 17 | S17 / H17 | S17 |
| Payout Blackjack | 3:2 / 6:5 | 3:2 |

---

## Dealer Peek

### ON

Con Peek attivo:

- se il dealer mostra un Asso viene proposta l'Insurance;
- se il dealer mostra una carta da 10 e possiede Blackjack naturale, la mano può essere risolta immediatamente.

### OFF

Con Peek disattivato:

- non viene proposta l'Insurance;
- il giocatore può eseguire le proprie azioni prima che il Blackjack naturale del dealer venga rilevato al normale momento di risoluzione.

---

## Soft 17

### S17

```text
Dealer stands on soft 17
```

Il dealer si ferma.

### H17

```text
Dealer hits soft 17
```

Il dealer pesca.

---

## Payout Blackjack

### 3:2

Profitto:

```text
puntata × 1,5
```

### 6:5

Profitto:

```text
puntata × 1,2
```

---

# Puntate e chips

## Saldo iniziale

Default:

```text
10.000 chips
```

Range configurabile:

```text
1.000 – 100.000
```

Incremento della configurazione:

```text
1.000
```

---

## Precisione

Le chips vengono normalizzate internamente a:

```text
1 cifra decimale
```

Questo permette di gestire correttamente:

- metà puntata;
- payout;
- Insurance;
- valori non interi derivati dalle regole.

---

## All-In

La UI permette di impostare direttamente:

```text
puntata = saldo disponibile
```

L'All-In dispone inoltre di un feedback audio dedicato.

---

## Ripeti puntata

È possibile riutilizzare l'ultima puntata compatibilmente con il saldo corrente.

Se il saldo è inferiore all'ultima puntata, il valore viene limitato alle chips disponibili.

---

# Oracolo

BlackNick include una funzione opzionale chiamata **Oracolo**.

L'host può:

```text
abilitarla
disabilitarla
```

Quando è abilitata, ogni giocatore può acquistare lo sblocco.

## Costo

Il costo è:

```text
chips iniziali × 2
```

Con configurazione standard:

```text
10.000 × 2 = 20.000 chips
```

Per acquistarlo il saldo deve essere superiore al costo.

---

## Funzionamento

L'Oracolo analizza la composizione dello **shoe residuo reale**.

La stima considera in particolare:

- quantità relativa di carte alte;
- quantità relativa di carte basse;
- Assi residui;
- carte da valore 10 residue;
- numero equivalente di mazzi rimasti.

La logica produce tre valori indicativi:

```text
Win
Loss
Push
```

Valori base:

```text
Win  ≈ 42%
Loss ≈ 49%
Push ≈ 9%
```

La composizione dello shoe modifica leggermente questi valori.

---

## Cosa NON è l'Oracolo

L'Oracolo:

- non esegue una simulazione Monte Carlo;
- non conosce il futuro ordine delle carte;
- non garantisce l'esito della mano;
- non rappresenta una probabilità Blackjack matematicamente completa;
- non sostituisce una strategia di base;
- non mostra esplicitamente la quantità di carte residue;
- non comunica direttamente quando lo shoe viene rigenerato.

È una **stima euristica indicativa** basata sulla composizione residua.

---

# Statistiche

BlackNick mantiene statistiche per la sessione corrente.

Sono tracciati:

- mani;
- vittorie;
- sconfitte;
- pareggi;
- Blackjack;
- percentuale vittorie;
- profitto massimo;
- saldo massimo;
- winning streak corrente;
- miglior winning streak.

---

## Cronologia

Vengono mantenute le ultime:

```text
10 mani
```

Per ogni voce sono disponibili:

- round;
- etichetta risultato;
- variazione di chips.

Esempi di etichette:

```text
VITTORIA
SCONFITTA
PAREGGIO
BLACKJACK
SPLIT
SURRENDER
```

Le statistiche non sono persistenti tra sessioni o rematch completi.

---

# Audio e feedback visivo

La V1.0 incorpora gli asset audio direttamente nell'HTML tramite Data URI Base64.

Non sono presenti URL remoti `.wav`, `.ogg` o `.mp3` per gli effetti e la musica del gioco.

---

## Musica

Tracce normali incluse:

```text
Late Night Casino
Jazz Lounge
Cyber Casino Ambient
```

Tracce dedicate alle streak:

```text
Pressure Pulse · STREAK 5
Overdrive · STREAK 10
```

Una traccia normale viene scelta per il match.

Il giocatore può cambiare localmente traccia quando non è attiva una musica streak.

---

## Winning streak

Quando la streak raggiunge:

```text
5 vittorie
```

viene utilizzata la traccia dedicata `STREAK 5`.

Quando raggiunge:

```text
10 vittorie
```

viene utilizzata `STREAK 10`.

La UI applica inoltre effetti visivi associati alla streak.

---

## Effetti sonori

Sono presenti effetti per:

- carte;
- fiche;
- vittoria;
- Blackjack;
- sconfitta;
- All-In.

I gruppi di effetti possono combinare più sample con piccoli delay per creare un suono più ricco.

---

## Web Audio

Parte del sistema utilizza:

```text
Web Audio API
```

Sono presenti anche fallback sintetici per alcuni effetti.

I browser possono impedire l'autoplay audio finché l'utente non interagisce con la pagina.

Per questo l'audio viene inizializzato dopo una prima interazione.

---

# Architettura

BlackNick utilizza un'architettura **single-file monolitica**.

```text
BN_V1.0.html
│
├── HTML
│   ├── Home
│   ├── Lobby
│   ├── Tavolo
│   ├── HUD
│   ├── Dock
│   ├── Pannelli
│   └── Overlay
│
├── CSS
│   ├── Tema
│   ├── Layout
│   ├── Responsive
│   ├── Animazioni
│   ├── Carte
│   ├── Glow
│   └── Streak FX
│
├── Asset incorporati
│   ├── Immagini Base64
│   ├── Musica Base64
│   ├── SFX Base64
│   └── Voce All-In Base64
│
└── JavaScript
    ├── Stato
    ├── Blackjack
    ├── Shoe
    ├── Puntate
    ├── Dealer
    ├── Split
    ├── Double
    ├── Surrender
    ├── Insurance
    ├── Oracolo
    ├── Statistiche
    ├── Audio
    ├── Rendering
    ├── PeerJS
    └── WebRTC
```

---

# Multiplayer P2P

## Modello host-authoritative

Nel multiplayer l'host mantiene lo stato principale.

Il client non modifica direttamente la partita.

Invia richieste come:

```text
hit
stand
split
double
surrender
insurance
selectHand
bet
cancelBet
buyOddsTool
endVote
```

L'host:

1. riceve il messaggio;
2. verifica che l'azione sia prevista;
3. verifica il turno e lo stato;
4. applica l'azione;
5. genera lo stato pubblico;
6. sincronizza lo stato ai client.

---

## Stato pubblico

Prima di inviare lo stato ai client, l'host crea una rappresentazione pubblica.

Durante:

```text
BETTING
INSURANCE
PLAYING
```

la seconda carta del dealer viene sostituita con:

```text
null
```

In questo modo la hole card non viene normalmente inviata ai client prima della fase in cui deve essere visibile.

---

## Room code

Formato:

```text
6 caratteri
```

Alfabeto:

```text
ABCDEFGHJKLMNPQRSTUVWXYZ23456789
```

Sono esclusi alcuni caratteri facilmente confondibili.

Esempio:

```text
7KMF2P
```

L'ID PeerJS dell'host viene costruito a partire dal codice.

---

## Limiti connessione

Configurazione corrente:

```text
MAX_PLAYERS       = 4
MAX_PENDING       = 8
HELLO_TIMEOUT     = 6000 ms
CONNECT_TIMEOUT   = 12000 ms
```

---

## Messaggi

Un messaggio ricevuto deve:

- essere un oggetto;
- non essere un array;
- possedere un campo `type` stringa;
- non superare circa 65.536 caratteri quando serializzato in JSON.

Le azioni principali ricevute dall'host vengono ulteriormente filtrate tramite whitelist.

---

## Watchdog host

L'host esegue un controllo periodico ogni:

```text
900 ms
```

Il watchdog prova a evitare che il gioco rimanga bloccato in alcune transizioni.

Può, ad esempio:

- avviare una mano quando tutte le puntate sono pronte;
- risolvere l'Insurance;
- avanzare oltre giocatori non attivi;
- riavviare un passo del dealer se il timer relativo non è più presente.

---

# Randomizzazione

La casualità importante del gioco utilizza:

```text
crypto.getRandomValues()
```

tramite Web Crypto API.

Viene utilizzata per:

- generazione dei codici stanza;
- shuffle dello shoe;
- selezioni casuali interne come la musica del match.

---

## Shuffle

Lo shoe viene creato e mescolato con una variante di:

```text
Fisher-Yates
```

La scelta dell'indice casuale utilizza `crypto.getRandomValues()` con rejection sampling per evitare bias semplice da modulo.

---

# Gestione dello shoe

Sono supportati:

```text
1
2
4
6
8
```

mazzi.

Numero iniziale di carte:

```text
deckCount × 52
```

---

## Esaurimento

Lo shoe viene rigenerato quando l'array delle carte risulta vuoto.

Non è presente una cut card o una percentuale configurabile di penetrazione.

### Conseguenza tecnica

Il controllo viene effettuato al momento della pesca.

Se lo shoe termina esattamente durante una sequenza di pescate, il codice può creare un nuovo shoe al successivo `drawCard()`.

Questo significa che la rigenerazione non è modellata come un evento fisico di casinò separato tra due mani.

È una semplificazione tecnica dell'implementazione corrente.

---

# Sicurezza applicativa

## Content Security Policy

Il documento include una CSP.

La V1.0 permette:

### Script

```text
'self'
'unsafe-inline'
https://unpkg.com
https://cdn.jsdelivr.net
```

### Connessioni

```text
'self'
https://0.peerjs.com
wss://0.peerjs.com
```

### Immagini

```text
'self'
data:
```

### Media

```text
'self'
data:
```

### Altre restrizioni

```text
object-src 'none'
base-uri 'none'
frame-ancestors 'none'
```

È inoltre presente:

```text
referrer = no-referrer
```

---

## Sanitizzazione nomi

I nomi giocatore:

- vengono convertiti in stringa;
- vengono ripuliti dai caratteri di controllo;
- vengono troncati;
- hanno una lunghezza massima applicativa.

---

## Trust model

Il modello di sicurezza è importante:

> **l'host è considerato affidabile.**

L'host possiede:

- lo shoe;
- l'ordine delle carte;
- la hole card del dealer;
- la logica di settlement;
- i saldi;
- lo stato della partita.

Il sistema è quindi adatto a partite tra persone che si fidano dell'host.

Non è un sistema:

- trustless;
- verificabile crittograficamente dai client;
- server-authoritative indipendente;
- anti-cheat professionale.

Un host che modifica il proprio file HTML potrebbe teoricamente modificare:

- lo shuffle;
- lo stato;
- i saldi;
- le carte;
- le regole;
- i messaggi inviati.

---

# Dipendenze di rete

Anche se gli asset del gioco sono incorporati, il multiplayer mantiene dipendenze esterne.

## PeerJS

Versione:

```text
1.5.5
```

PeerJS viene caricato dinamicamente.

Fallback:

```text
https://unpkg.com/peerjs@1.5.5/dist/peerjs.min.js
https://cdn.jsdelivr.net/npm/peerjs@1.5.5/dist/peerjs.min.js
```

Se il primo CDN non funziona, viene tentato il secondo.

---

## Signaling

Il client utilizza il servizio PeerJS predefinito.

La CSP autorizza:

```text
https://0.peerjs.com
wss://0.peerjs.com
```

---

## ICE / STUN

Configurazione:

```text
stun:stun.l.google.com:19302
stun:stun1.l.google.com:19302
```

Non è configurato un TURN server dedicato.

---

# Self-contained: cosa significa realmente

La V1.0 è **self-contained per gli asset applicativi**.

Sono incorporati nel documento:

- grafica;
- icone;
- musica;
- effetti sonori;
- voce All-In;
- CSS;
- JavaScript applicativo.

La CSP media consente infatti soltanto:

```text
'self'
data:
```

---

## Cosa funziona senza server di asset

Il gioco non deve più scaricare da Internet:

- effetti carte;
- effetti fiche;
- effetti vittoria;
- effetti sconfitta;
- effetti Blackjack;
- musica;
- immagini incorporate.

---

## Cosa richiede ancora Internet

La V1.0 richiede rete per il normale flusso PeerJS:

- download della libreria PeerJS, se non già disponibile;
- registrazione del peer;
- signaling;
- connessione multiplayer;
- uso degli STUN esterni.

Inoltre, poiché il Solo viene avviato dopo la creazione dell'host PeerJS, **la modalità Solo standard non è completamente indipendente dalla rete**.

Quindi:

```text
Self-contained assets ≠ completamente offline
```

---

# Requisiti

## Browser

Serve un browser moderno con supporto a:

- JavaScript ES6+;
- WebRTC;
- Web Crypto API;
- Web Audio API;
- CSS Custom Properties;
- CSS animations;
- Data URI;
- `backdrop-filter` per la resa grafica completa.

---

## Browser consigliati

- Chrome / Chromium recente.
- Microsoft Edge recente.
- Firefox recente.
- Safari recente.

Il progetto è principalmente orientato all'utilizzo desktop.

---

## Rete

Per il multiplayer:

- connessione Internet;
- accesso a PeerJS;
- WebSocket/HTTPS non bloccati;
- WebRTC consentito;
- NAT/firewall compatibile oppure connettività P2P possibile.

---

# Avvio

## Metodo consigliato: server HTTP locale

Posiziona il file in una cartella.

Apri un terminale nella stessa cartella.

Con Python:

```bash
python -m http.server 8000
```

Poi apri:

```text
http://localhost:8000/
```

Se il file si chiama:

```text
BN_V1.0.html
```

apri:

```text
http://localhost:8000/BN_V1.0.html
```

---

## Nome consigliato per distribuzione

Per un hosting statico è possibile rinominare il file:

```text
index.html
```

senza modificare la logica interna.

---

## Apertura diretta

È possibile provare ad aprire direttamente il file `.html`.

Esempio:

```text
file:///.../BN_V1.0.html
```

Tuttavia l'esecuzione tramite `localhost` è generalmente preferibile perché:

- evita alcune restrizioni del contesto `file://`;
- rende il comportamento più simile a un vero hosting;
- facilita il debugging;
- riduce differenze tra browser.

---

# Come giocare

## Creare una stanza

1. Apri BlackNick.
2. Inserisci il nome.
3. Premi `Crea stanza`.
4. Attendi l'apertura del peer.
5. Copia il codice stanza.
6. Condividilo con gli amici.
7. Configura le regole.
8. Avvia Solo oppure attendi altri giocatori.

---

## Entrare in una stanza

1. Apri BlackNick.
2. Inserisci il nome.
3. Inserisci il codice stanza.
4. Premi `Entra`.
5. Attendi handshake e sincronizzazione.

---

## Avviare il multiplayer

L'host può avviare il match quando sono presenti almeno:

```text
2 giocatori
```

---

## Puntare

Durante `BETTING`:

1. seleziona l'importo;
2. usa eventualmente i pulsanti rapidi;
3. usa `All In` se desiderato;
4. conferma la puntata.

La mano parte quando i giocatori richiesti hanno confermato.

---

# Flusso di una partita

Fasi principali:

```text
WAITING
   ↓
BETTING
   ↓
INSURANCE      se applicabile
   ↓
PLAYING
   ↓
DEALER
   ↓
DONE
   ↓
BETTING
```

Quando il match termina:

```text
MATCH OVER
```

---

## WAITING

- ingresso giocatori;
- configurazione regole;
- attesa dell'avvio.

---

## BETTING

- scelta delle puntate;
- conferma;
- sincronizzazione;
- partenza automatica quando tutti sono pronti.

---

## INSURANCE

Compare solamente quando previsto dalle regole.

Ogni giocatore idoneo deve:

- accettare;
- rifiutare;
- oppure viene considerato automaticamente pronto se non dispone dei fondi necessari.

---

## PLAYING

I giocatori agiscono in ordine.

Il sistema salta automaticamente giocatori:

- fuori dal match;
- con Blackjack;
- già in Stand;
- non attivi.

---

## DEALER

Il dealer completa la propria mano secondo:

```text
S17
oppure
H17
```

Se nessuna mano valida richiede il confronto con il dealer — per esempio perché tutte le mani sono già in Bust o Surrender — il dealer non continua a pescare inutilmente.

---

## DONE

Vengono mostrati:

- risultato;
- variazione saldo;
- statistiche;
- overlay;
- effetti;
- eventuale streak.

Successivamente è possibile procedere con la mano seguente.

---

# Fine del match

Nel multiplayer un giocatore con:

```text
0 chips
```

viene eliminato.

Il match continua finché restano almeno due giocatori con saldo positivo.

---

## Un solo giocatore rimasto

Se rimane un solo giocatore con chips:

```text
quel giocatore vince
```

---

## Nessun giocatore rimasto

Caso limite:

se tutti gli ultimi giocatori terminano contemporaneamente le chips:

```text
vince il dealer
```

---

## Fine consensuale

I giocatori attivi possono votare per terminare il match.

Se tutti i giocatori richiesti accettano:

- il match viene chiuso;
- vince il giocatore con il saldo maggiore;
- in caso di parità al primo posto viene dichiarata la situazione di pari saldo.

---

## Rematch

Il rematch:

- ripristina le chips iniziali;
- resetta statistiche;
- resetta mani;
- crea un nuovo shoe;
- sceglie una nuova musica normale;
- riporta la partita alla lobby.

---

# Disconnessioni

Le disconnessioni vengono gestite dall'host.

Il giocatore disconnesso viene rimosso dallo stato.

---

## Durante un match multiplayer

Se la disconnessione lascia un solo giocatore attivo:

```text
il giocatore rimasto può essere dichiarato vincitore
```

---

## Connessione con l'host persa

Il client torna alla lobby mostrando un messaggio dedicato.

Non è presente un sistema completo di:

- resume della sessione;
- reconnect con recupero dello stesso player;
- migrazione dell'host.

---

## Broker PeerJS disconnesso

Il client prova a chiamare:

```text
peer.reconnect()
```

Una DataChannel P2P già stabilita può in alcuni casi continuare a funzionare anche se il broker di signaling si scollega.

Non è però garantito il recupero completo in tutti gli scenari.

---

# Limiti tecnici

Questa sezione descrive volutamente i limiti della versione corrente.

---

## 1. Host trusted

Il principale limite architetturale è il modello host-authoritative locale.

Non esiste un server terzo che certifichi:

- shuffle;
- carte;
- saldi;
- vincitori.

Per partite private è una scelta semplice ed efficace.

Per un contesto competitivo non è sufficiente.

---

## 2. Nessun TURN server

Sono configurati STUN pubblici ma non TURN.

Con alcune reti:

- CGNAT;
- NAT simmetrico;
- firewall aziendali;
- reti universitarie;
- hotspot restrittivi;
- VPN;

la connessione P2P potrebbe fallire.

---

## 3. Dipendenza da PeerJS pubblico

La creazione della stanza dipende dalla disponibilità:

- della libreria PeerJS via CDN;
- del servizio di signaling utilizzato da PeerJS.

Se questi servizi sono irraggiungibili, il multiplayer non parte.

---

## 4. Solo non completamente offline

Gli asset sono locali, ma la modalità Solo standard viene raggiunta tramite la creazione dell'host PeerJS.

Senza PeerJS disponibile l'interfaccia corrente non inizializza normalmente la stanza Solo.

---

## 5. Nessuna persistenza

Un refresh della pagina può perdere:

- match;
- saldo;
- statistiche;
- stanza;
- connessioni;
- cronologia.

Non è presente salvataggio in:

- database;
- cloud;
- account;
- backend.

---

## 6. Nessuna host migration

Se l'host lascia la partita:

```text
la stanza non viene trasferita automaticamente a un altro peer
```

---

## 7. Nessun reconnect stateful completo

Un client che perde la connessione non dispone di un protocollo completo per:

- riconoscersi come stesso giocatore;
- recuperare la vecchia mano;
- riprendere il turno;
- riottenere automaticamente lo stesso saldo.

---

## 8. Shoe semplificato

Non sono presenti:

- cut card;
- penetration;
- burn card;
- reshuffle point configurabile.

Il nuovo shoe viene creato quando quello corrente risulta vuoto.

---

## 9. Re-split non supportato

Il codice supporta uno Split iniziale ma non una catena arbitraria di re-split.

---

## 10. Regole non completamente configurabili

Alcune regole sono fisse.

Tra queste:

```text
Double After Split = ON
Split Aces One Card = ON
```

Non sono esposte come toggle della lobby.

---

## 11. Surrender semplificato

Il Surrender è implementato come azione disponibile su una mano iniziale non splittata di due carte.

Non è presente una modellazione separata e configurabile delle varianti da casinò come:

- Early Surrender;
- Late Surrender.

---

## 12. Oracolo euristico

Le percentuali dell'Oracolo sono stime interne.

Non devono essere interpretate come probabilità rigorose o garantite.

---

## 13. File molto grande

La V1.0 contiene asset Base64 direttamente nel documento.

Dimensione indicativa:

```text
~7,8 MB
```

Vantaggi:

- portabilità;
- nessuna cartella asset;
- nessun link audio esterno;
- copia semplice.

Svantaggi:

- editor più lenti;
- diff Git molto pesanti;
- Base64 poco leggibile;
- merge conflict difficili;
- memoria maggiore durante il parsing;
- repository più pesante.

---

## 14. Codice monolitico

HTML, CSS, JavaScript e asset convivono nello stesso file.

Questo riduce le dipendenze locali ma rende più difficile:

- manutenzione;
- testing;
- code review;
- refactoring;
- collaborazione tramite Git.

---

## 15. CDN JavaScript ancora esterni

Il file è self-contained per gli asset, ma PeerJS non è incorporato.

I CDN restano quindi un punto di dipendenza.

---

## 16. Nessun matchmaking

Le stanze sono private e basate su codice.

Non esistono:

- elenco stanze;
- ricerca giocatori;
- lobby pubbliche;
- matchmaking.

---

## 17. Nessun sistema account

I nomi dei giocatori non identificano un account reale.

Non sono presenti:

- password;
- login;
- profilo;
- identità persistente.

---

## 18. Nessuna protezione anti-modifica

Essendo un HTML locale:

```text
chi possiede il file può modificarlo
```

Questo è coerente con la natura del progetto, ma impedisce di trattarlo come client competitivo affidabile.

---

# Troubleshooting

## La stanza non viene creata

### Sintomi

- `Creo la stanza...` resta bloccato;
- compare timeout;
- PeerJS genera un errore;
- il codice non viene visualizzato.

### Controllare

1. Connessione Internet.
2. Console browser.
3. Accesso a `unpkg.com`.
4. Accesso a `cdn.jsdelivr.net`.
5. Accesso a `0.peerjs.com`.
6. Firewall.
7. VPN.
8. WebSocket.
9. Estensioni privacy/ad blocker.

### Prova rapida

Aprire DevTools:

```text
F12 → Console
```

e verificare errori relativi a:

```text
PeerJS
CSP
WebSocket
ERR_BLOCKED
ERR_CONNECTION
```

---

## "Libreria multiplayer non caricata"

Significa che entrambi i tentativi di caricare PeerJS non sono riusciti.

CDN tentati:

```text
unpkg
jsDelivr
```

Possibili cause:

- assenza di rete;
- CDN bloccato;
- CSP modificata;
- estensione browser;
- DNS;
- proxy;
- firewall.

---

## "Timeout nella creazione della stanza"

Il timeout applicativo è circa:

```text
12 secondi
```

Il peer non è riuscito ad aprirsi entro quel periodo.

Provare:

- ricaricare;
- cambiare rete;
- disattivare temporaneamente VPN;
- usare un browser Chromium recente;
- usare localhost anziché `file://`.

---

## "Codice stanza non valido"

Il codice deve:

- avere esattamente 6 caratteri;
- utilizzare solo l'alfabeto previsto.

Formato:

```text
ABCDEFGHJKLMNPQRSTUVWXYZ23456789
```

Non aggiungere:

- spazi;
- trattini;
- prefissi.

---

## "Stanza non trovata o host non raggiungibile"

Possibili cause:

- codice errato;
- host ha chiuso la pagina;
- stanza terminata;
- peer ID non più registrato;
- signaling non disponibile;
- rete dell'host incompatibile;
- firewall;
- NAT restrittivo.

---

## "Timeout: stanza non raggiungibile"

Il client è riuscito ad aprire PeerJS ma non a stabilire la connessione con l'host nel tempo previsto.

Controllare soprattutto:

- host ancora online;
- codice corretto;
- NAT/firewall;
- VPN;
- rete mobile;
- blocchi WebRTC.

---

## "Handshake scaduto"

L'host ha accettato una connessione in ingresso, ma non ha ricevuto correttamente il messaggio iniziale entro:

```text
6 secondi
```

Può indicare:

- connessione instabile;
- client modificato;
- messaggi non consegnati;
- errore WebRTC;
- versione incompatibile.

---

## "Handshake non valido"

Il primo messaggio del client non rispetta il protocollo atteso.

L'host richiede un messaggio:

```text
type = hello
```

con un nome valido.

Probabile in presenza di:

- file modificati;
- versioni incompatibili;
- client non ufficiale.

---

## "Handshake duplicato"

Un peer ha inviato nuovamente `hello` dopo essere già stato accettato.

Questo non fa parte del normale flusso.

---

## "Peer già presente"

La stanza contiene già un peer con lo stesso ID.

Può accadere in scenari anomali di:

- riconnessione;
- duplicazione;
- stato PeerJS non ancora scaduto.

---

## "Stanza piena"

La stanza supporta al massimo:

```text
4 giocatori
```

---

## "Match già iniziato"

Nella modalità multiplayer normale non è possibile entrare arbitrariamente in un match già in corso.

Il join è accettato principalmente nella lobby.

La modalità Solo possiede una gestione separata per l'arrivo di un amico.

---

## Il multiplayer funziona su una rete ma non su un'altra

È uno dei limiti più comuni di WebRTC P2P senza TURN.

Esempio:

```text
Wi-Fi casa     → funziona
rete aziendale → non funziona
```

Possibile causa:

```text
NAT / firewall
```

STUN non garantisce la connessione in ogni topologia.

---

## Il broker PeerJS si scollega ma la partita continua

È possibile.

Una volta creata la connessione WebRTC, i peer possono possedere già un DataChannel diretto.

Il broker serve soprattutto al signaling.

Il gioco tenta comunque una riconnessione al broker.

---

## La connessione con l'host si chiude

Il client non può continuare autonomamente perché l'host mantiene lo stato autoritativo.

La UI torna quindi alla lobby.

---

## Il dealer non pesca dopo che tutti hanno sballato

È intenzionale.

Se nessuna mano rimasta richiede un confronto:

```text
il dealer non deve completare inutilmente la propria mano
```

Il round viene direttamente risolto.

---

## Il dealer pesca su 17

Controllare la regola:

```text
S17 / H17
```

Con H17 il dealer pesca solamente sul **soft 17**.

Un hard 17 resta Stand.

---

## Non compare Insurance

Controllare:

1. Dealer Peek deve essere ON.
2. La carta scoperta deve essere un Asso.
3. Il giocatore deve essere idoneo.
4. Il saldo deve permettere l'Insurance per poterla acquistare.

---

## Non compare Split

Lo Split richiede:

- esattamente due carte;
- stesso rank;
- saldo sufficiente;
- mano non già splittata.

Esempio:

```text
K + Q
```

non è splittabile nell'implementazione corrente.

---

## Non compare Double

Servono:

- due carte nella mano corrente;
- puntata valida;
- saldo sufficiente per duplicare la puntata.

---

## Non posso fare re-split

È un limite dell'implementazione corrente.

Dopo lo Split iniziale non viene consentita una nuova divisione delle mani split.

---

## Dopo Split degli Assi non posso continuare

È intenzionale.

La regola implementata è:

```text
Split Aces → una carta aggiuntiva → Stand
```

---

## Il Blackjack dopo uno Split non paga 3:2

È intenzionale.

Il payout Blackjack naturale viene applicato soltanto alla mano originale di due carte.

Le mani split vengono trattate come normali mani da 21.

---

## L'audio non parte

I browser possono bloccare l'autoplay.

Provare:

1. cliccare nella pagina;
2. aumentare il volume FX;
3. aumentare il volume musica;
4. controllare che la scheda browser non sia muta;
5. controllare l'output audio del sistema.

---

## Alcuni effetti si sovrappongono

È possibile per design.

Alcuni eventi usano gruppi di sample con delay leggermente diversi.

Questo crea effetti stratificati, soprattutto per:

- fiche;
- vittorie;
- Blackjack.

---

## La musica cambia durante una streak

È previsto.

A:

```text
5 vittorie consecutive
```

entra la musica `STREAK 5`.

A:

```text
10 vittorie consecutive
```

entra `STREAK 10`.

Durante una streak il cambio manuale del brano viene limitato.

---

## La pagina è pesante da aprire nell'editor

La V1.0 è un file di circa 7,8 MB e contiene grandi stringhe Base64.

Editor o IDE possono:

- usare molta RAM;
- rallentare la syntax highlighting;
- rallentare il diff.

Per il browser questo non implica necessariamente un problema, ma per lo sviluppo può essere scomodo.

---

## Git mostra diff enormi

È una conseguenza dell'inclusione Base64.

Una piccola modifica a un asset può generare una stringa molto lunga modificata.

---

## L'Oracolo mostra percentuali strane

Le percentuali sono volutamente:

```text
stime indicative
```

Non rappresentano un calcolo esatto dell'EV della mano corrente.

Vengono derivate dalla composizione dello shoe residuo.

---

## L'Oracolo non mostra quante carte restano

È intenzionale.

Il tool utilizza il mazzo residuo internamente ma non espone direttamente:

- numero carte;
- conteggio;
- momento esatto del reshuffle.

---

## Dopo un refresh ho perso tutto

È previsto.

Non c'è persistenza del match.

Un refresh ricrea il contesto JavaScript e interrompe:

- PeerJS;
- WebRTC;
- stato;
- saldi;
- statistiche.

---

## Il gioco resta fermo in una fase

L'host dispone di un watchdog interno ogni circa 900 ms.

Se rimane comunque fermo:

1. aprire la Console;
2. controllare eventuali errori JavaScript;
3. verificare lo stato della connessione;
4. verificare se l'host ha chiuso/sospeso la scheda;
5. evitare throttling aggressivo del browser;
6. ricaricare come ultima opzione.

Un refresh interromperà il match corrente.

---

# Debug e test interni

Il file espone alcune utility di test tramite `window`.

Tra quelle presenti:

```text
testBlackjackLogic()
testDealerPeekRule()
testCasinoRules()
```

Possono essere eseguite dalla Console del browser.

Esempio:

```javascript
testCasinoRules()
```

---

## Debug deck control

È presente un hook interno per il controllo del deck in fase di debug.

Di default:

```text
enabled = false
```

Il normale gameplay non utilizza carte forzate.

Questo meccanismo è pensato per facilitare test deterministici della logica.

---

# Struttura del progetto

La versione documentata è composta principalmente da:

```text
BN_V1.0.html
README.md
```

Non sono necessarie cartelle locali per gli asset audio o grafici incorporati.

---

## Filosofia single-file

### Vantaggi

- elevata portabilità;
- un solo documento da inviare;
- nessuna dipendenza relativa da cartelle asset;
- ridotto rischio di link locali rotti;
- facile archiviazione;
- avvio semplice.

### Svantaggi

- file molto grande;
- manutenzione più difficile;
- Base64 poco leggibile;
- merge Git complessi;
- impossibilità di caching separato degli asset;
- qualsiasi modifica richiede una nuova copia del documento completo.

---

# Compatibilità

## Desktop

Il progetto è pensato soprattutto per browser desktop moderni.

La resa migliore richiede supporto completo alle API e agli effetti CSS utilizzati.

---

## Mobile

Il layout può adattarsi in parte, ma il progetto non va considerato principalmente mobile-first.

Browser mobili possono inoltre applicare:

- limiti audio più aggressivi;
- sospensione delle schede;
- throttling timer;
- restrizioni WebRTC;
- comportamenti differenti sui DataChannel.

---

## Browser vecchi

Non sono supportati browser privi di:

```text
WebRTC
Web Crypto
Web Audio
ES6+
```

---

# Privacy e persistenza

BlackNick non implementa un sistema account o un database applicativo.

I dati di partita vengono mantenuti principalmente:

```text
in memoria nel browser
```

e sincronizzati attraverso WebRTC durante la sessione.

---

## Dati tipicamente presenti nello stato

- nome giocatore;
- peer ID;
- carte;
- puntate;
- saldo;
- statistiche;
- stato partita;
- regole.

---

## Persistenza

Non è presente una persistenza applicativa strutturata tra sessioni.

Chiudendo o ricaricando la pagina lo stato corrente può essere perso.

---

## Server applicativo

Non è presente un backend BlackNick dedicato che memorizzi:

- account;
- password;
- saldi;
- partite;
- cronologia globale.

Il servizio PeerJS esterno viene comunque utilizzato per il signaling di rete.

---

# Licenza

Nel file HTML documentato non è dichiarata una licenza software generale del progetto.

Prima di pubblicare il repository è consigliabile aggiungere un file:

```text
LICENSE
```

con la licenza scelta dal proprietario del progetto.

Gli eventuali asset di terze parti incorporati devono essere utilizzati nel rispetto delle rispettive condizioni di licenza, indipendentemente dalla licenza applicata al codice BlackNick.

---

# Disclaimer

BlackNick è un progetto software di intrattenimento e sperimentazione.

Utilizza:

```text
chips virtuali
```

e non integra denaro reale.

Non offre:

- depositi;
- prelievi;
- vincite monetarie;
- pagamenti;
- wallet;
- conversione chips-denaro.

Le stime dell'Oracolo sono puramente indicative e non costituiscono garanzie matematiche o finanziarie.

---

# Riepilogo tecnico

| Voce | Implementazione |
|---|---|
| Tipo | Single-file web application |
| Frontend | HTML / CSS / JavaScript vanilla |
| Backend applicativo | Nessuno |
| Multiplayer | PeerJS + WebRTC |
| Modello rete | Host-authoritative P2P |
| Giocatori | Max 4 |
| Room code | 6 caratteri |
| Default chips | 10.000 |
| Mazzi | 1 / 2 / 4 / 6 / 8 |
| Blackjack | 3:2 / 6:5 |
| Soft 17 | S17 / H17 |
| Split | Sì |
| Re-split | No |
| DAS | Sì |
| Split Aces | Una carta |
| Surrender | Sì |
| Insurance | Con Dealer Peek |
| Oracolo | Opzionale |
| Statistiche | Sessione |
| Cronologia | Ultime 10 mani |
| Audio | Base64 incorporato |
| Immagini | Incorporate |
| PeerJS | 1.5.5 via CDN |
| STUN | Google |
| TURN | Non configurato |
| Persistenza | No |
| Account | No |
| Anti-cheat trustless | No |
| Denaro reale | No |

---

<p align="center">
  <strong>BlackNick</strong><br>
  Blackjack · P2P · PeerJS · WebRTC · Vanilla JavaScript
</p>
