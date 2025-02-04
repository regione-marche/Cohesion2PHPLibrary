# Cohesion2 libreria PHP
Libreria per l'autenticazione al sistema di SSO Cohesion2 della Regione Marche. 
Questa libreria  permette di integrare il  Single Sign-On di Cohesion2 in siti o applicativi web sviluppati in linguaggio PHP. 

## Requisiti di installazione
* PHP 5
* Nel file php.ini  assicurarsi che il parametro allow_url_fopen sia impostato a On

## Installazione
copiando la directory cohesion2 in un qualsiasi punto della cartella web dell’applicativo. Assicurarsi che la cartella contenga il seguente file:
-	Cohesion2.php


## Abilitazione SSO
Il Single Sign-On è abilitato per default nella libreria. Questo significa che prima di reindirizzare l’utente alla maschera di login, il sistema verifica la validità della sessione ed evita quindi all’utente di doversi riautenticare.
Per disabilitare, eventualmente il SSO, e forzare quindi sempre all’autenticazione, utilizzare il seguente comando:

```php
      $cohesion = new Cohesion2;
      $cohesion->useSSO(false);
      $cohesion->auth();
```

### Esempio di utilizzo

```php
      require_once 'cohesion2/Cohesion2.php';
      try{
          $cohesion = new Cohesion2;
          $cohesion->auth();
      }
      catch(Exception $e){
          die($e->getMessage());
      }
      if($cohesion->isAuth()){
          echo 'Utente autenticato: '.$cohesion->username.'<br>';
          echo 'Id SSO: '.$cohesion->id_sso.'<br>';
          echo 'Id Aspnet: '.$cohesion->id_aspnet.'<br>';
          echo 'Profilo: <pre>'.var_export($cohesion->profile,1).'</pre>';
      } 
```

## Abilitazione SAML 2.0
E' possibile indicare a Cohesion di utilizzare lo standard SAML 2.0 tramite l'apposito metodo **useSAML20()** . L'utilizzo di tale metodo permette agli utenti di autenticarsi anche tramite sistema SPID.

```php
      $cohesion = new Cohesion2;
      $cohesion->useSAML20(true);
      $cohesion->enableEIDASLogin(); //Inserire solo se si intende abilitare il login eIDAS
      $cohesion->enableSPIDProLogin(array("PF", "PG", "LP")); //RIMUOVI per disabilitare il login SPID Professionale
      $cohesion->auth();
```


## Spiegazione del meccanismo di autenticazione
Invocando il metodo auth() della classe Cohesion2 viene avviato il processo di autenticazione tramite SSO. Il processo si svolge in 4 passi:

1. Viene invocata la pagina web https://cohesion2.regione.marche.it/sso/Check.aspx per verificare se l’utente risulti già autenticato tramite SSO
2. Nel caso l’utente non sia autenticato, il browser dell’utente viene automaticamente reindirizzato alla pagina di login https://cohesion2.regione.marche.it/SA/AccediCohesion.aspx
3. Se l’autenticazione ha esito positivo, la libreria istanzia una variabile di sessione  per tenere traccia dell’avvenuta autenticazione ed invoca la pagina web https://cohesion2.regione.marche.it/SSO/webCheckSessionSSO.aspx per recuperare il profilo dell’utente autenticato
4. Se il recupero del profilo è avvenuto correttamente i dati dell’utente saranno accessibili tramite le seguenti proprietà dell’oggetto istanziato:
- `$cohesion->username` (Username utente autenticato)
- `$cohesion->id_sso` (ID della sessione SSO)
- `$cohesion->id_aspnet` (ID della sessione ASPNET)
- `$cohesion->profile` (Array contenente il profilo della persona)

Per la configurazione SAML 2.0 il funzionamento è analogo, cambiano solamente gli endpoint utilizzati e le possibilita di accesso per l'utente (SPID, CIE-ID, CNS, Cohesion, eIDAS, ...)

## Profilo utente autenticato
Tramite la proprietà  *profile*  è possibile accedere ai dati del profilo utente. I campi disponibili sono quelli forniti dal 
sistema Cohesion e vengono istanziati come chiavi dell’array profile (non tutti i campi possono risultare valorizzati): 
*titolo, nome, cognome, sesso, login, codice_fiscale, spidCode, telefono, localita_nascita, provincia_nascita, cap_nascita, regione_nascita,
data_nascita, nazione_nascita, gruppo, ruolo, email, mail_certificata, telefono_ufficio, fax_ufficio, numero_cellulare, 
indirizzo_residenza, localita_residenza, provincia_residenza, cap_residenza, regione_residenza, nazione_residenza, professione, 
settore_azienda, profilo_familiare, tipo_autenticazione (PW,CF)*.

Esempio:

```php
      echo $cohesion->profile['nome'].' '.$cohesion->profile['cognome'];
```

## Procedura di logout
Per chiudere la sessione locale e disconnettere l’utente dal sistema di SSO utilizzare il metodo logout():

```php
      $cohesion = new Cohesion2;
      $cohesion->logout();
```

L’esempio completo è visualizzabile nel file *test/logout.php* o nel file *test/logout_saml20.php*
Per chiudere, eventualmente, solo la sessione locale lasciando aperta quella del SSO utilizzare il metodo logoutLocal():

```php
      $cohesion = new Cohesion2;
      $cohesion->logoutLocal();
```

## Limitazione dei metodi di autenticazione permessi
Il metodo setAuthRestriction  permette di limitare i metodi di autenticazione permessi .

```php
      $cohesion->setAuthRestriction('1,2,3');
```

I valori 0,1,2,3 indicano i livelli di autenticazione da mostrare nella pagina di login Cohesion:

- 0 = autenticazione con Utente e Password
- 1 = autenticazione con Utente, Password e PIN
- 2 = autenticazione con Smart Card
- 3 = autenticazione di Dominio (valida solo per utenti interni alla rete regionale)

E’ possibile nascondere o visualizzare le modalità di autenticazione togliendo o aggiungendo i rispettivi valori separati da una virgola. L’ordine è ininfluente.

N.B. Se si intende limitare l’accesso in base al tipo di autenticazione, è necessario, oltre ad utilizzare tale metodo, inserire un controllo per gli utenti che risultino già autenticati in SSO.

```php
      if($cohesion->profile['tipo_autenticazione']!='PW'){
          echo ‘OK puoi usare il servizio’;
      }
      else echo ‘Autenticazione debole non permessa’;
```
