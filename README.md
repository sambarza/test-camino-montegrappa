# Scopo
Ho un caminetto montegrappa ad area forzata, voglio pilotare la velocità della ventola tramite app.

# Funzionamento originale
La velocità della ventola è pilotata da una centralina che ha in ingresso il sensore della temperatura e in uscita il connettore per la ventola. Più la temperatura misurata dal sensore è alta più la velocità della ventola è alta, quando la temperatura è bassa, la ventola è spenta.

Il sensore è un termistore NTC (codice ricambio 1042001100, sonda 6x60mm, cavo 220cm) montato nel condotto dell'aria calda. Più la temperatura è alta, più la resistenza è bassa.
Misura nota: 85 Ω a 30°C. La curva R/T completa (coefficiente beta) è da determinare con una seconda misura a temperatura diversa.

# Idea
L'idea è di fare un bridge fisico tra il sensore esterno della temperatura e la centralina.
Questo bridge è un circuito comandato da un ESP8266, in ingresso si connette il sensore originale, in uscita si connette la centralina del camino.
Il circuito simula resistenze diverse verso la centralina tramite un potenziometro digitale pilotato dall'ESP8266, che in cascata comandano velocità diverse della ventola.

# Funzionalità

## Modalità automatica
La velocità della ventola è comandata dalla temperatura misurata dal sensore originale.
Il circuito connette direttamente il sensore originale con la centralina.

## Modalità manuale
La velocità della ventola è impostata manualmente tramite app, 5 velocità da minimo a massimo.
Il circuito interpone il potenziometro digitale tra l'ESP8266 e la centralina.

# Sicurezza
Il circuito è fail-safe: se l'ESP8266 non è alimentato o si blocca (crash), entra automaticamente in modalità automatica collegando il sensore originale direttamente alla centralina.

Quando l'ESP8266 è operativo, la soglia di sicurezza è configurabile tramite app. Se la temperatura supera la soglia, il circuito attiva automaticamente la modalità automatica indipendentemente da quella selezionata.

Il fail-safe è implementato tramite un watchdog hardware (timer 555 in modalità monostabile): l'ESP8266 invia un impulso periodico (heartbeat, ~20ms ogni 500ms); se il timer scade senza ricevere l'impulso (timeout ~2s), forza il circuito in modalità automatica.

# Accensione
Al momento dell'accensione il circuito si configura in modalità automatica.

# Schema del circuito

Lo schema elettrico è in `hardware/` (progetto KiCad, da completare).

## Commutazione segnale

Due photorelay (componenti solid-state, fail-safe) realizzano la commutazione SPDT:
- **PR1 (NC, normalmente chiuso)**: in serie con il sensore NTC — di default connesso
- **PR2 (NO, normalmente aperto)**: in serie con il potenziometro digitale — di default disconnesso

```
Sensore NTC ────[PR1 NC]────┐
                             ├──── Centralina
Potenziometro ──[PR2 NO]────┘
```

Stato a riposo (photorelay non eccitati): PR1 chiuso, PR2 aperto → sensore NTC → centralina (modalità automatica).

## Controllo photorelay (AND implicito)

```
VCC ──[555 WD output]──[R1]──┬── PR1 LED ──┐
                              └── PR2 LED ──┤
                                            └── Collettore NPN
                 ESP8266 mode ──[R2]──────── Base NPN
                                             Emettitore → GND
```

I LED dei photorelay si accendono solo se entrambe le condizioni sono vere:
- 555 output HIGH (watchdog soddisfatto, ESP8266 vivo)
- ESP8266 mode GPIO HIGH (modalità manuale selezionata)

## Watchdog (555 monostabile)

```
ESP8266 ──► heartbeat (impulso 20ms ogni 500ms) ──► 555 trigger

Finché arriva heartbeat: 555 output = HIGH → photorelay possono eccitarsi
Heartbeat assente > ~2s:  555 output = LOW  → photorelay de-eccitati → AUTO
```

## LED pannello (hardware, indipendente da ESP8266)

Un transistor PNP pilotato dall'uscita NPN inverte il segnale di controllo:
- Photorelay eccitati (manuale) → LED spento
- Photorelay de-eccitati (automatica, anche per watchdog) → LED acceso

# Architettura del sistema
Il sistema è composto da tre elementi:
- **hardware/** — circuito stampato (schema KiCad)
- **firmware/** — firmware ESP8266: legge il sensore NTC, pilota il potenziometro digitale, gestisce il heartbeat watchdog, espone API REST
- **app/** — app di controllo: seleziona modalità, imposta velocità manuale, configura soglia di sicurezza
