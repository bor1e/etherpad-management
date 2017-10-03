# Etherpad mit SSO Anbindung und Benutzeroberfläche für private Pads


## Etherpad
Es wird die Version 1.5.7 verwendet.

Anleitung der Installation unter [Etherpad-Lite](www.github.com/ether/etherpad-lite)

`settings.json` für interne Einstellungen des Servers (Anbindung an port, Datenbank, etc) zu verwenden, als Beispiel Vorlage dient `settings.json.template`.

Es wurde der MySQL Server des RRZE verwendet (in `settings.json`):

` "dbType" : "mysql",
  "dbSettings" : {
                    "user"    : "etherpad",
                    "host"    : "mysqlrz.db.fau.de",
                    "port"    : 3306,
                    "password": "our-password-for-db",
                    "database": "etherpad_prod"
                  },
`Sollte man alte/bereits existierende Pads exportieren/integrieren wollen, muss man sich zuerst einen MySQL-dump erstellen, und diesen in die zuintegrierende Datenbank einfügen [mehr dazu](https://github.com/ether/etherpad-lite/wiki/Backing-up-and-Restoring-Etherpad-Lite-Pads).

## Benutzeroberfläche
Es wurde `ep_user_pad` und `ep_user_pad_frontend` als [Web-Frontend](https://github.com/aoberegg/ep_user_pad_frontend/) verwendet.
Installation kann entweder über das etherpad-admin tool oder direkt über den Link durchgeführt werden. Wichtig hierbei auch die Datenbank [entsprechend](https://github.com/aoberegg/ep_user_pad_frontend/blob/master/installation.pdf) zu initialisieren.

Die Darstellung der Webseiten erfolgt dynamisch unter Verwendung der [Bootstrap](http://bootstrapdocs.com/v3.3.6/docs/getting-started/)-Bibliothek Version `3.3.6` und befindet sich in dem `bootstrap/`-Verzeichnis (nicht mehr in dem `/templates`-Verzeichnis).
Die Auflistung der Gruppen und Pads erfolgt mithilfe der Javascript Bibliothek [Datatables](https://datatables.net/).

##Vorbereitung zur SSO Anbindung

Folgende Bibliotheken mussten nach installiert werden.
* `npm install body-parser`
* `npm install express-session`
* `npm install method-override`
* `npm install ms`
+ `npm install passport-saml`

Diese wurden in erster Linie zur Verwendung von `passport-saml` genutzt.
Es muss auch ein `express`-Update auf eine Version `4.x.x` durchgeführt werden.
**Wichtig: sowohl `ep_user_pad` als auch `ep_user_pad_frontend` werden seid 14.02.14 nicht mehr gewartet. Dies führt zu Problem mit gemeinsam genutzen Bibliotheken von `etherpad` und `passport-saml`. Insbesondere hat sich die fundamentale Bibliothek `express` stark verändert und verwendet die von dem `frontend` erwartenden Bibliotheken nicht entsprechend.**

**Achtung: es wird ein `router`- eine spezielle Express-Verbindung für die SSO für `passport-saml` verwendet.**

## Datenbank
Bei der Datenbank ist die 'uid' Spalte hinzuzufügen, um die entsprechende FAU-Kennung mitabzuspeichern:

`
use 'etherpad_prod';
ALTER TABLE User ADD uid varchar(32) NOT NULL;
ALTER TABLE USER ADD UNIQUE (uid);
`

## SSO Anbindung
Anbindung mittels `passport-saml`-Modul.
Installation entsprechend der [README](https://github.com/bergie/passport-saml/).

An das SSO der Uni angebunden wird es in `etherpad/etherpad-lite/ep_user_pad_frontend/hooks.js` ab Zeile 441:

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

Stand: 03. Oktober 2017
