# EWE Glasfaser mit OpenWRT und eigenem Modem
Mit diesem Post möchte ich den Stand der Glasfaseranbindung mit eigenem Router und Modem über EWE (in der Glasfaser-Nordwest Geschmacksrichtung) vom März 2024 festhalten.

### Disclaimer:
Einfacher ist die Einrichtung des Anschlusses mit einer fritz!Box fiber. Hier passiert alles automatisch.

### Modem:
Ich habe mich für ein gebrauchtes [Telekom Glasfasermodem 2](https://geizhals.de/telekom-glasfaser-modem-2-40823382-a2601735.html) entschieden, da ich eventuell später den Router wechsele und dann eine einfache Anbindung über Ethernet an den meist vorhandenen WAN Port des neuen Geräts stattfinden kann. Das Gerät ist ein GPON Gerät, was zum Netz der Glasfaser-Nordwest passt. Man kann auf dem Webinterface des Modems einige Verbindungsinfos einsehen, dies ist aber voraussichtlich nicht nötig. Man kann dort auch ein PLOAM Passwort einstellen, was man bei der EWE aber nicht benötigt.

- Besorge ein Glasfasermodem.
- Stecke das Modem mit dem beiligenden Glasfaserkabel in die frisch installierte Glasfaserdose.
- Verbinde das Modem per Ethernetkabel mit dem Port des Routers der (später) als WAN Port konfiguriert wird.
- Schalte das Modem am Schalttag ein: Blinkt es, war die Schaltung noch nicht erfolgreich. Leuchtet es durchgehend, hat es sich mit dem Netz verbunden und Dein Router kann eine PPPoE Verbindung aufbauen (siehe unten).

**Wichtig: Modem-ID**
```Die EWE (über GFNW) authorisiert das Modem über seine Modem-ID. Diese steht auf der Rückseite des Modems. Gib die Modem-ID (nicht die Seriennummer) dem technischen Support von EWE. Die Telefonnummer findest Du auf dem Brief, auf dem Dir die Zugangsdaten mitgeteilt wurden. Dort steht etwas kleiner ein Absatz zum Thema: "eigenes Endgerät".```

- Gib der EWE per Telefon die Modem-ID.

### Router
- Besorge einen Router.
- Installiere OpenWRT und verbinde Dich mit der Weboberfläche.
- Finde heraus, welcher Port dein WAN Port wird und wie er heißt (eth2 in meinem Fall).

Unter OpenWRT fasst die Bridge "br-lan" Ethernetports zusammen, die auf der Seite des LAN hängen. Wir wollen einen Port als WAN Port, daher müssen wir den aus der br-lan herausnehmen und die anderen hinzufügen.

- Network -> Interfaces -> Devices -> br-lan -> Configure -> Bridge ports -> Entferne den Port, der später zum WAN Port wird und füge die hinzu, die Du auf der LAN Seite möchtest.

![[img_1.png]]

Damit können wir das WAN Interface konfigurieren. Die Zugangsdaten stehen im Brief der EWE. Die EWE benutzt VLAN 7, dies wird auch im Brief mitgeteilt. Um ein VLAN einzustellen kann man im WAN Interface ein "custom device" angeben, dessen Name aus dem WAN Port (eth2) und einem Punkt (.) gefolgt von der VLAN ID (7) besteht, also "eth2.7" in meinem Fall. Dies funktioniert auch, wenn das VLAN device vorher nicht konfiguriert wurde. Man kann auch extra ein VLAN device vorher konfigurieren, was ich aber nicht getan habe. Es taucht dann nach der WAN Konfiguration unter "Devices" auf.

- Notiere Dir den "custom device" Namen passend zu Deiner Konfiguration (nameDesWanPorts.7), Du brauchst ihn gleich.

Das vorkonfigurierte WAN Interface kann gelöscht werden. Auch das wan6 Interface kann gelöscht werden, da EWE keine IPv6 zur Verfügung stellt (Stand: Februar 2024).

- Network -> Interfaces -> wan -> Delete
- Network -> Interfaces -> wan6 -> Delete
- Network -> Interfaces -> Add new Interface
	- Name: Wähle einen kreativen Namen, z.B. "Glasfaser"
	- Protocol: PPPoE
	- Device: unten bei "custom device" eth2.7 (**DEIN** WAN Port mit ".7" am Ende)
	- -> Save
- Daraufhin öffnet sich ein Fenster mit weiteren Einstellungen. Wenn nicht, klicke beim neuen PPPoE Interface auf "Configure".
	- Reiter "General Settings":
	- PAP/CHAP username: Deine Benutzerkennung aus dem Brief
	- PAP/CHAP password: Dein Passwort aus dem Brief
	- Reiter "Firewall Settings":
	- Create/Assign Firewall Zone -> wan
	- -> Save

![[img_2.png]]
![[img_3.png]]

Jetzt sollte alles funktionieren. Wenn jedoch keine Verbindung aufgebaut wird, kannst Du im OpenWRT log die Fehlermeldungen nachlesen.

Danke an das [Glasfaserforum](https://glasfaserforum.de)!
