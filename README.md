# Etherpad mit SSO Anbindung und Benutzeroberfläche für private Pads


## Etherpad
Es wird die Version 1.5.7 verwendet. (Aktuellste Version von etherpad-lite)

Anleitung der Installation unter [Etherpad-Lite](www.github.com/ether/etherpad-lite)

`settings.json.template` verwenden um interne Einstellung (Anbindung an port, Datenbank, etc) zu verwenden, anschließend als `settings.json` abspeichern.

MySQL-Beispiel:

` "dbType" : "mysql",
  "dbSettings" : {
                    "user"    : "etherpad",
                    "host"    : "localhost",
                    "port"    : "/var/run/mysqld/mysqld.sock",
                    "password": "your-password-for-db",
                    "database": "etherpad-lite"
                  },
`

## Benutzeroberfläche
Es wurde `ep_user_pad` und `ep_user_pad_frontend` als [Web-Frontend](https://github.com/aoberegg/ep_user_pad_frontend/) verwendet.
Installation kann entweder über das etherpad-admin tool oder direkt über den Link durchgeführt werden. Wichtig hierbei auch die Datenbank [entsprechend](https://github.com/aoberegg/ep_user_pad_frontend/blob/master/installation.pdf) zu initialisieren. 
Bei der Datenbank ist die 'uid' Spalte hinzuzufügen:

`
mysql -u root -p
use 'etherpad-lite'
ALTER TABLE User ADD uid varchar(32) NOT NULL;
ALTER TABLE USER ADD UNIQUE (uid);
`

Folgende Bibliotheken mussten nach installiert werden.
* `npm install body-parser`
* `npm install express-session`
* `npm install method-override`
* `npm install ms`

Diese wurden in erster Linie zur Verwendung von `passport-saml` genutzt. 
Es muss auch ein `express`-Update auf eine Version `4.x.x` durchgeführt werden.
**Wichtig: sowohl `ep_user_pad` als auch `ep_user_pad_frontend` werden seid 14.02.14 nicht mehr gewartet. Dies führt zu Problem mit gemeinsam genutzen Bibliotheken von `etherpad` und `passport-saml`. Insbesondere hat sich die fundamentale Bibliothek `express` stark verändert und verwendet die von dem `frontend` erwartenden Bibliotheken nicht entsprechend.**

Die Darstellung der Webseiten wurde entsprechend in `templates/` angepasst. 
Dabei wurde
* `login/logout` verlinkt
* Manipulation an der Datenbank durch Benutzer unterbunden

## SSO Anbindung
Anbindung mittels `passport-saml`-Modul.
Installation entsprechend der [README](https://github.com/bergie/passport-saml/).

An das SSO der Uni angebunden:

` 
var samlStrategy = new saml.Strategy({
    callbackUrl: 'https://etherpad.rrze.uni-erlangen.de/pass/login/callback',
    entryPoint: 'https://www.sso.uni-erlangen.de/simplesaml/saml2/idp/SSOService.php',
    issuer: 'https://etherpad.rrze.uni-erlangen.de/metadata',    //issuer string to supply to identity provider
    }, function(profile, done) {
      var user = {};
      user.uid = profile['urn:mace:dir:attribute-def:uid'];
      user.email = profile['urn:mace:dir:attribute-def:mail'];
      user.displayName = profile['urn:mace:dir:attribute-def:displayName'];
      user.username = profile['urn:mace:dir:attribute-def:givenName']
      user.name = profile['urn:mace:dir:attribute-def:sn']
      registerUser(user, function(userAdded, finished){
	if(userAdded){
	  return done(null, user);
	} else {
          sendError('Failed to Add User to DB');
        }
      });
  });
`

**Achtung: es wird ein `express.Router()` für die Anbindung verwendet. Dies trennt sauber `body-paser` (verwendet für `passport-saml`) und `formidable` (verwendet für `ep_user_pad_frontend`)  von einander und verhindert ungewolltes Verhalten.**

Stand: 17. August 2015
