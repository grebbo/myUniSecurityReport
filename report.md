% MyUniversity Security Report
% Federico D'Ambrosio; Edoardo Ferrante; Enrico Ferro

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
risorse, come file e cartelle, a lui precluse.

# Vulnerabilità

## Content Provider
MyUniversity dispone di un content provider il quale agisce come interfaccia tra applicazioni esterne e la nostra app al fine di modificare le tabelle del database.

Idealmente il content provider dovrebbe dare accesso ad una fantomatica applicazione dell'unige per poter aggiornare ad ogni semestre gli orari in nostro possesso; in pratica il content provider non richiede alcuna autenticazione da parte dell'app di terzi e perciò concede ad ogni app installata sul dispositivo di cambiare orari di lezioni ed esami, cambiare le aule degli stessi e persino estrarre informazioni sulla carriera dello studente.

### Possibili attacchi

Un agente tramite applicazione malevola potrebbe cambiare data e ora di lezioni e/o esami, per poter così direzionare l'utente in un certo luogo e ad una certa ora per intenti criminosi, potrebbe altresì cancellare o modificare la data di un esame per far sì che lo studente non vi si presenti rovinandogli così la carriera accademica (per esempio facendo posticipare la laurea).

In pratica l'attacco, posto che l'attaccante conosca la struttura del database e le funzioni del content provider si esegue semplicemente chiamando letture tramite query e/o modifiche tramite update sugli stessi dati.

### Contromisure

Per evitare che app di terzi (potenzialmente malevoli) possano accedere e modificare i dati nel database possiamo semplicemente definire una nuova tipologia di permesso, necessaria per poter accedere al nostro content provider: l'utente verrebbe così avvisato e dovrebbe dunque accettare che la suddetta app possa accedervi.

Definisco un permesso richiedente firma.

```xml
<permission android:name="com.grebeteam.myuniversity.provider.IO"
android:protectionLevel="signature" />
```

In questo modo solo applicazioni aventi nel proprio Manifest

```xml
<uses-permission android:name="com.grebeteam.myuniversity.provider.IO"/>
```
e recanti la stessa firma di MyUniversity potranno accedere al content provider, rimuovendo così la vulnerabilità.

La dichiarazione del content provider diventa, quindi

```xml
 <provider
	android:name=".MyContentProvider"
	android:authorities="com.grebeteam.myuniversity.MyContentProvider"
	android:exported="true"
	android:permission= "com.grebeteam.myuniversity.provider.IO"
/>
```

## Esportazione/importazione database per il backup

All'interno dell'applicazione è presente una funzionalità per l'esportazione e l'importazione del database della stessa, in modo da fornire all'utente un backup locale. L'attuale implementazione dell'esportazione
crea il file `/MyUniversity/exportedDB.sqlite` nella memoria interna del telefono (`/storage/emulated/0/`). L'importazione, in maniera duale rispetto
all'esportazione cerca il file `/MyUniversity/exportedDB.sqlite` e lo sostituisce completamente al database in uso (da evidenziare che è possibile importare il database unicamente nella fase di primo setup dell'applicazione).
Il file, come detto, si trova in una cartella pubblica ed è facilmente accessibile, quindi, da altre applicazioni, anche malevole, che potrebbero sia leggere, sia scrivere su di esso, senza alcun problema. 

### Possibili attacchi

Il possibile attacco che può essere effettuato sfruttando la vulnerabilità appena citata è il seguente:

1. L'utente effettua il backup (esportazione) del database;

2. L'attaccante legge o modifica il database a proprio piacimento (in caso di sola lettura, la privacy dell'utente è a rischio);

3. L'utente ripristina l'applicazione e importa il database di backup, ormai modificato dall'attaccante;

### Contromisure

Per rendere più sicura l'esportazione/backup del database, è possibile sfruttare le **Backup API** di Google, funzioni apposite per il backup. Queste API sono 2: **Key/Value Backup**, necessario per i device con Android 5.1 o inferiore, e 
**AutoBackup** per i dispositivi con Android 6.0 o superiore. MyUniversity è compilata avendo come target e versione minima compatibile Android 6.0, per cui è necessario implementare solamente l'aggiunta di AutoBackup. 

Questi backup vengono caricati, se di dimensione minore a 25MB, sul cloud di Google Drive dell'utente, senza che questo debba effettuare alcuna azione.

### AutoBackup {-}

Per implementare AutoBackup si aggiungono gli attributi

```xml
<application ...
	android:allowBackup="true"
	android:fullBackupContent="@xml/my_backup_rules">
```
all'AndroidManifest.xml, dove `my_backup_rules` è il file posizionato in res/xml/my_backup_rules.xml contenente le regole riguardanti i file da escludere o includere nel backup.

```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <include domain="database" path="."/>
</full-backup-content>
```

Se volessimo estendere il supporto per il backup a device con Android 5.1 o inferiore, sarebbe necessario implementare, previa registrazione dell'applicazione all'[Android Backup Service](https://developer.android.com/google/backup/index.html), un apposito `BackupAgent`, 
che tenga traccia dello stato dei backup e abbia opportune implementazioni delle funzioni `onBackup()` e `onRestore()`.

\pagebreak

## SQL Injection

Una vulnerabilità spesso trovata in applicazioni che si interfacciano con database è l'SQL Injection. Durante l'analisi di sicurezza, abbiano accertato che MyUniversity non è a rischio di tale vulnerabilità in quanto ogni interazione con il database (insert, update e delete) avviene tramite PreparedStatement appositi per database SQLite:
`SQLiteStatement`. 

Per esempio:
```java
public int addGrade(Course course){

        String UPDATE = "update " + TABELLA_CORSI + " set " + VOTO + "= ? , " 
					+ DATA_ESAME + "= ? where codice = " + course.getCodice();

        //Statement per una query protetta da SQL Injection
        SQLiteStatement updateStatement = myDB.compileStatement(UPDATE);

        updateStatement.bindString(1, Integer.toString(course.getVoto()));
        updateStatement.bindString(2, course.getData_esame());

        return updateStatement.executeUpdateDelete();
}
```

## Logging

Una tipica vulnerabilità che si può riscontrare su sistemi Android precedenti 4.1 riguarda l'analisi dei log file, 
contenenti messaggi riguardanti il codice utili in fase di debugging dell'applicazione stessa. L'accesso a tale risorsa può fornire ad un agente malevolo informazioni sensibili dell'utente o 
riguardo l'esecuzione del software previa chiamata `Runtime.getRuntime().exec("logcat")`. 
MyUniversity è compatibile solo con devices con Android 4.4 o superiore, risolvendo questo problema su ogni dispositivo *unrooted*.  

## Sicurezza nelle connessioni remote

Per garantire la riservatezza e l'autenticazione delle richieste http di myUniversity alle risorse remote dei server Google, viene utilizzato il client delle API di Google che implementa il protocollo [OAuth2](https://tools.ietf.org/html/rfc6749) per l'autenticazione, 
insieme a HSTS (HTTP Strict Transport Security), per garantire autenticazione, integrità e riservatezza delle informazioni scambiate con i server di Google, durante la
procedura di login (e seguente download di immagini di profilo e copertina) e l'interazione con Google Calendar e Google Maps, 
eccezion fatta per il calcolo del percorso dalla posizione corrente alla destinazione, la cui richiesta viene fatta tramite HTTPS e viene inviato dal server codificato in base64.

```java
//Login tramite account Google
GoogleSignInOptions gso = 
	new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
        .requestEmail()
        .build();

GoogleApiClient mGoogleApiClient = new GoogleApiClient.Builder(this)
    .enableAutoManage(this, this)
    .addApi(Auth.GOOGLE_SIGN_IN_API, gso)
    .build();

private void signIn() {
    Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
    startActivityForResult(signInIntent, RC_SIGN_IN);
}

//Calcolo percorso tramite richiesta HTTPS a Google Maps

String myreq = "https://maps.googleapis.com/maps/api/directions/json?origin=" 
						+ urls[0] + "&destination=" + urls[1] + "&mode=" + urls[2];
						
try (CloseableHttpClient httpClient = HttpClientBuilder.create().build()) {
    CloseableHttpResponse response = httpClient.execute(new HttpGetHC4(myreq));
    BufferedReader rd = new BufferedReader(
    new InputStreamReader(response.getEntity().getContent()));
    result = new StringBuilder();
    String line = "";
    while ((line = rd.readLine()) != null) {
        result.append(line);
    }
} catch (Exception e) {
	e.printStackTrace();
}

```


