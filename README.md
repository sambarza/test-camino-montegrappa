# Scopo
Ho un caminetto montegrappa ad area forzata, voglio pilotare la velocità della ventola tramite app.

# Funzionamento originale
La velocità della ventola è pilotata da una centralina che ha in ingresso il sensore della temperatura 1042001100 e in uscita il connettore per la ventola.
Più la temperatura misurata dal sensore è alta più la velocità della ventola è alta, quando la temperatura è bassa, la ventola è spenta.
Il sensore esterno temperatura è implementato tramite il sensore XXX, più la temperatura è alta, più la resistenza è bassa.

# Idea
L'idea è di fare un bridge fisico tra il del sensore esterno della temporatura e la centralina.
Questo bridge è un circuito comandato da un ESP8266, in ingresso si connette il sensore originale, in uscita si connette la centralina del camino.
Il circuito fisico realizza un potenziometro elettronico pilotato dall'ESP8266, il circuito simula temperature diverse che ha cascata comandano velocità diverse della ventola.

# Funzionalità
Le funzionalità implementate dal bridge sono le seguenti.

## Modalità automatica
La velocità della ventola è automatica comandanta dalla temperatura misurata dal sensore originale.
Il circuito connette direttamente il sensore originale con la centralina.

## Modalità manuale
La velocità della ventola è in modalità manuale, 5 velocità, da minimo a massimo, non c'è una funzionalità per spegnere completamente la ventola.
Il circuito regola la resistenza del potenziometro.

# Sicurezza
In qualsiasi modalità sia impostato il circuito, se la temperatura misurata dal sensore originale supera i XXX gradi Celsius, il circuito attiva automaticamente la modalità automatica.

# Accensione
Al momento dell'accensione il circuito si configura in modalità automatica.

# Architettura del sistema
Il sistema è composto da quattro elementi:
- il circuito stampato
- il firmware dell'ESP8266, che pilota il circuito ed espone dei servizi REST
- l'app