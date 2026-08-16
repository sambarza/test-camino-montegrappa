# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Progetto

Bridge fisico per il caminetto Montegrappa ad area forzata: un circuito basato su ESP8266 si inserisce tra il sensore di temperatura originale e la centralina, simulando un potenziometro elettronico per controllare la velocità della ventola via app.

## Struttura del repo

- `hardware/` — schemi e layout del circuito stampato (PCB)
- `firmware/` — firmware ESP8266; espone API REST per controllare la modalità e la velocità della ventola
- `app/` — app di controllo (framework non ancora scelto)

## Architettura

Il firmware ESP8266 è il nodo centrale: legge il sensore di temperatura originale, pilota il potenziometro elettronico verso la centralina, ed espone API REST che l'app consuma. La logica di sicurezza (override automatico sopra soglia di temperatura) risiede nel firmware e non può essere disabilitata dall'app.

## Comportamento all'accensione

Il sistema parte sempre in **modalità automatica** (sensore originale → centralina, senza intermediazione del potenziometro).

## Git

- Email commit: `sambarza@gmail.com`
- Configurata solo a livello di repo locale (non globale)
