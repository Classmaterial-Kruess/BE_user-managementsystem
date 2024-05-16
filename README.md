# User Management System
In diesem Fullstack Projekt wird ein User Management System erstellt.<br>
Es umfasst Authentifizierung, Authorisierung sowie die Anzeige und Manipulation von User Einträgen.

## Features
Das System hält persistente Usereinträge mit einer Zuweisung in der Datenbank. Der API-Server stellt dabei die Schnittstelle zum Auslesen, Anlegen, Bearbeiten und Entfernen von den Einträgen dar.<br>
Zusätzlich soll ein User-Rollensystem für die Zugangsberechtigungen eingebaut werden.

## API-Server
Als API-Server wird eine REST Schnittstelle (bzw. RPC) auf Basis einer `express.js` Anwendung gebaut.<br>
Für den komfortablen Umgang mit einer `MongoDB` Datenbank wird das `Mongoose` _ORM_ als Middleware verwendet.<br>
Um die Authentifizierung der Clientanfragen nach einem initialen Login komfortabel aber sicher zu halten, wird ein `JSON Web Token (JWT)` verwendet.<br>
Da es mittlerweile zum guten Sicherheitsstandart gehört, sollen Passwörter der Nutzer __NICHT__ in Klarschrift in der Datenbank gespeichert werden. Zum _Hashen_ und _Salten_ empfiehlt sich der Einsatz von `bcrypt`.


## Frontend Client
Als Client kommt eine im Browser ausgeführte Webanwendung zum Einsatz. Diese soll mit `React.js` entwickelt werden.<br>
Die Frontendanwendung ist als Mensch-Maschine Schnittstelle einer modernen Client-Server Anwendung zu verstehen. Idealerweise beinhaltet sie möglichst wenig Businesslogik. Sie bedient größtenteils die _API_ und stellt dabei gelieferte Daten dar, bietet die nötigen Formulare zum Anlegen und Bearbeiten von Einträgen sowie alle nötigen Bedienfelder für weitere Interaktionen.
> Die Anwendung soll vor allem funktional und übersichtlich sein. Ansonsten wird freier Gestaltungsraum gegeben.

Für eine komfortable HTTP-Kommunikation mit der _API_ empfiehlt sich die Nutzung von `axios.js`.<br>
Eventuell sollte auch über ein globales Zustandsmanagement (`Context API`, `Redux` oder `Zustand.js`) nachgedacht werden.


## Datenstruktur
Da es sich um ein User Management System handelt, ist der Aufbau von Usereinträgen der zentrale Bestandteil der gesamten Datenstruktur.

### Aufbau Usereinträge
```
- ID: ObjectId 
    - sollte automatisch von der DB vergeben werden
- username: string
    - sollte eindeutig innerhalb der DB sein
- password: string 
    - gehasht und gesaltet in der DB
- location: string 
    - eine Stadt, ein Bürostandort oder eine Abteilung
- role: number
    - Die Referenz auf die dem User zugeordnete Userrolle (andere Collection), die seine Nutzungberechtigungen darstellt.
```

### Aufbau Userrollen
Die Userrollen mit den ihnen zugeordneten Nutzungsberechtigungen sollten für eine bessere Wartbarkeit ebenfalls in der Datenbank gespeichert werden. So ist es zukünftig recht einfach, die Berechtigungen bestimmter Rollen zu erweitern oder zu beschränken ohne einer Neuauslieferung des Servercodes.
```
- ID: ObjectId 
    - sollte automatisch von der DB vergeben werden
- label: string
    - stellt den Namen der Rolle dar (z.B. 'Admin')
- privilegeLevel: number
    - stellt eine Ordnungszahl für die Hierarchie der Userrollen dar
- canRead: boolean
    - regelt, ob die Rolle Usereinträge anfordern und auslesen darf
- canWriteSelf: boolean
    - regelt, ob Nutzer mit dieser Rolle ihre eigenen Einträge verändern dürfen
- canWriteOthers: boolean
    - regelt, ob Nutzer mit dieser Rolle die Einträge anderer Nutzer abändern dürfen
```

Folgende Userrollen können bereits standardmäßig angelegt werden:
- Regular user -> der normale _eingeloggte_ Nutzer
    - privilegeLevel: 3
    - canRead: true
    - canWriteSelf: true
    - canWriteOthers: false
    - canDelete: false
- Moderator -> Rolle mit leicht erhöhten Berechtigungen
    - privilegeLevel: 2
    - canRead: true
    - canWriteSelf: true
    - canWriteOthers: true
    - canDelete: false
- Administrator -> Rolle mit maximalen Berechtigungen
    - privilegeLevel: 1
    - canRead: true
    - canWriteSelf: true
    - canWriteOthers: true
    - canDelete: true
- Deactivated user -> Rolle für gesperrte oder gelöschte Nutzer im Schwebezustand bevor der Eintrag komplett entfernt wird
    - privilegeLevel: 4
    - canRead: false
    - canWriteSelf: false
    - canWriteOthers: false
    - canDelete: false

> Das Feld `privilegeLevel` wird verwendet, um eine Hierarchie der Userrollen darzustellen. Das wird nützlich, wenn es darum geht, ob z.B. ein Moderator den Eintrag eines Administrators bearbeiten kann.<br>
Folgende Regel ist dabei abzuleiten: Wenn `privilegeLevel` des Zieleintrags kleiner als das eigene, dann sind Schreibzugriffe erlaubt, sonst nicht.

## API Endpoints
Neben einem Endpoint für `/login`, der zur Authentifizierung des Users und dem Ausstellen eines Tokens genutzt wird,
sollten die üblichen __CRUD__ Endpoints zum _Abrufen_, _Hinzufügen_, _Editieren_ und _Entfernen_ eines Usereintrags gestellt werden.
> Beachte dabei die entsprechenden Zugangsberechtigungen!

Es kann auch ein `GET`-Endpoint für Userrollen angelegt werden. Von weiteren __CRUD__ Endpoints wird an dieser Stelle aufgrund der Komplexität des Projekts abgesehen.

> __Für alle Endpoints gilt:__ Vom Server gesendete Statuscodes sollten so sinnvoll wie möglich nach _REST_ zurückgegeben werden. Mache dir grundsätzlich gedanken darüber, wann und in welcher Form Daten vom Server an den Client zurück geschickt werden sollten bzw. was im Fehlerfall zurückgesendet werden soll.

### Authorisierung
Stelle sicher, dass alle Endpoints abgesehen vom `/login` Endpoint nur mit einem gültigen `JWT` Token zugänglich sind. Dafür könnte eine zentrale _Middlewarefunktion_ geschrieben werden, die vor jeden _Requestcontroller_ vorgeklemmt werden kann.
Es gilt zu überlegen, ob der Token per _Request Payload_ via `JSON` oder per _httpOnly Cookie_ ausgeliefert werden soll.

## Projekt- und Dateistruktur
Es gilt: __Clean code__.<br>
Überlege dir von vorne rein eine saubere und übersichtliche Ordnerstruktur für die beiden Projekte.<br>

Neben dieser README findest du im Projekt noch zwei weitere Ordner jeweils für den _Server_ und den _Client_.<br>
Das sind jeweils eigene Projektordner, erstelle das `npm`-Projekt für den React-Client im Ordner `client` und das `npm`-Projekt für den Express-Server im Ordner `server`.<br>
Die darin enthaltenen `placeholder.txt` Dateien können gelöscht werden. Sie dienen lediglich als Platzhalter damit `git` die leeren Ordner nicht ignoriert.