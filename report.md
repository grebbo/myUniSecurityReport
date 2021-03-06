# MyUniversity Security Report {.unnumbered}

## Vulnerabilità e messa in sicurezza dell’applicazione MyUniversity {.unnumbered}

# Introduzione e premesse

MyUniversity è un’applicazione Android sviluppata per supportare lo studente
durante la carriera universitaria, fornendo vari strumenti come la prenotazione
degli esami, calendario delle lezioni.

La seguente relazione sulle vulnerabilità di sicurezza trovate all’interno
dell’applicazione si pone di evidenziare e tappare quelle falle del software che
potrebbero essere usate da un agente malevolo, che **non** abbia i permessi di
root all’interno del dispositivo in cui è installata MyUniversity.

I permessi di root, infatti, renderebbero la presente analisi inadeguata, visto
che un agente dotato di tali permessi potrebbe accedere senza restrizioni a
risorse, come ad esempio file e cartelle, che, normalmente, gli sarebbero
preclusi.

In aggiunta, si segnala che viene utilizzato il client delle API di Google che implementa il protocollo [OAuth2](https://tools.ietf.org/html/rfc6749) per l'autenticazione, 
insieme a HSTS (HTTP Strict Transport Security), per garantire autenticazione, integrità e riservatezza delle informazioni scambiate con i server di google, durante la
procedura di login (e seguente download di immagini di profilo e copertina) e l'interazione con Google Calendar e Google Maps, 
eccetto il calcolo del percorso dalla posizione corrente alla destinazione, la cui richiesta viene fatta tramite HTTPS e viene inviato dal server codificato in base64.

# Vulnerabilità conosciute

## Content Provider

## Esportazione/importazione database per il backup

## Analisi dei log

# Possibili attacchi

## Content Provider

## Esportazione/importazione database per il backup

## Analisi dei log

# Contromisure

## Content Provider

## Esportazione/importazione database per il backup

## Analisi dei log

