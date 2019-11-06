# Baue deine eigene Cloud

### Ablauf:
Zeit | Programmpunkt
-- | --
17:00 | Was ist das für ein Gerät? Was ist Nextcloud?
17:30 | Den Odroid zusammenbauen
18:00 |Das Netzwerk einrichten
18:30 | Nextcloud installieren
20:00 | Pizza und Theorie: DynDNS und Port Forwarding
21:00 | Ende (Eventuell wirds auch 22:00)

### Inhalt:
Odroid HC2, Gehäuse, Stromkabel + Anschluss EU, LAN Kabel, SD Card, Festplatte

### Lämpchen und deren Bedeutung:
* Rotes Lämpchen (PWR): Das Gerät hat Strom
* Blaues Lämpchen, blinkend (ALIVE): Das Gerät ist in Betrieb.
* Grünes Lämpchen, leuchtend: Auf die Festplatte wird zugegriffen.

### Benötigte Software auf dem Computer:
* Windows: Putty ist ein Bash Terminal, mit welchem wir uns mit dem Gerät verbinden. Alternativ kann unter Windows 10 auch der Linux Kernel verwendet werden
* Mac/Ubuntu: Terminal

### Verwendete Befehle:
* `ssh`: (secure shell) Verbindungsherstellung zu einem (anderen) Gerät im Netzwerk
* `sudo`: (superuser do) Befehlsvorsatz, welcher den nachfolgenden Befehl mit erhöhten Privilegien ausführt
* `apt update`: Herunterladen der aktuellen Software-Liste
* `apt upgrade`: Upgrade aller installierter Software auf die neueste Version
* `apt install`: Installiert neue Software von der Software-Liste
* `passwd`: Passwort ändern
* `adduser`: Erstellt einen neuen Nutzer
* `usermod`: Verwaltet die Nutzer Gruppen Beziehungen
* `cd`: (change directory) Im Verzeichnisbaum navigieren
* `ls`: (list) alle Unterverzeichnisse und Dateien im aktuellen Verzeichnis darstellen
* `fdisk`: (format disk) Festplatte für Dateisystem vorbereiten
* `mkfs.ext4`: (make file system) Festplatte mit EXT4 Dateisystem formatieren
* `mkdir`: (make directory) Neues Verzeichnis erstellen
* `mount`: Datenträger ins Verzeichnis einbinden
* `blkid`: stellt die verfügbaren Speicherallokationen dar
* `nano`: Konsolen-Editierprogramm (Beenden mit STRG+X)
* `wget`: (web get) lädt eine Datei herunter.
* `chmod`: Nutzerrechte für eine Datei ändern
* `tar`: Komprimierungsprogramm um Dateiarchive zu erstellen und zu entpacken

### IT-Konzepte:
* root
* Verzeichnisbaum
* partition
* filesystem
* server (hardware, software)

### Step by Step
0. Gerät in Betrieb nehmen: Bereits mit einer UBUNTU Installation vorbereitet ist die SD Karte. Soll das System neu installiert werden, kann die UBUNTU Installation unter [1] bezogen werden. Die ISO Datei muss dazu auf den USB Stick “gebrannt” werden. Unter Windows geschieht das mit dem Tool XXX

1. Netzwerk verwalten: Beim Verbinden des Geräts mit dem Router wird automatisch eine Netzwerkaddresse zugewiesen. Diese kann zum Beispiel über das Webinterface des Routers herausgefunden werden. Den Laptop verbinden wir mit dem WLAN `techlabwlan`. Passwort `12341234`

2. Gerät Konfigurieren: 
	1. Auf das Gerät wird mittels ssh zugegriffen. Das ist möglich mit dem Programm Putty (Download), über den Windows 10 Linux Kernel (Featureverwaltung), oder über das Terminal bei Mac und Ubuntu. Der Befehl zum verbinden lautet `ssh root@<<ip-addresse>>`. Die IP wurde in Schritt 1 ermittelt. Das Passwort zu diesem Zeitpunkt lautet `odroid`.
	2. Als erstes sollte das System upgedated werden `apt update` lädt dabei die aktuellsten Software Listen herunter. Anschliessend wird `apt upgrade` ausgeführt um das System zu aktualisieren.
	3. Um das System vor ungewünschtem Eindringen zu schützen muss das root geändert werden. Mit dem Befehl `passwd` wird die Änderung des Passworts des eingeloggten Users eingeleitet. Das Passwort sollte möglichst hart sein, und wird nur selten verwendet. (Empfehlung: Zufallskombination + Klebezettel auf dem Gerät). 
	4. Zur sichereren Bedienung wird ein weiterer Benutzer angelegt. Dies geschieht mit dem Befehl `adduser <<username>>`. Dieser Nutzer wird unser Standartnutzer sein. Bei Linux/Unix Systemen wird empfohlen den gleichen Nutzernamen zu wählen. Mit `usermod -aG sudo <<username>>` wird der Nutzer der sudo Gruppe hinzugefügt, womit die Ausführung mit privilegierten Rechten möglich wird.
	5. Um das Gerät (in unserem Setting) eindeutig identifizierbar zu machen wird mit `sudo hostnamectl set-hostname <<name>>` der Gerätename geändert.
	6. Die Verbindung mit dem Gerät wird mit `exit` getrennt. Anschliessend verbinden wir uns wieder mit unserem neu angelegten Nutzer: `ssh <<username>>@<<ip-address>>`

3. Festplatte bereitstellen:
	1. Zunächst wird mit `sudo fdisk -l` die Festplatten-ID ermittelt. Anschliessend wird das Formatierungstool aufgerufen mit `sudo fdisk <<ID>>` (beim odroid ist ID immer `/dev/sda`). Die Hilfe zum Tool kann mit `m` (menu) ausgegeben werden. Um die Festplatte zu formatieren wählen wir `n` (new partition) aus. Im Dialog legen wir den primary Partition Type fest (`p`) und bestätigen die Standardwerte der restlichen Eingaben. Mit `w` (write) werden die Änderungen übernommen. Anschliessend kann die Partition mit `sudo fdisk -l` geprüft werden.
	2. nach der Partitionierung muss anschliessend das Filesystem bereitgestellt werden. Für das EXT4 Filesystem passiert das mit dem Befehl `sudo mkfs.ext4 /dev/sda1` (ID vorhergehend ermittelt).
	3. Anschliessend wird die Festplatte in den Verzeichnisbaum eingebunden. Mit `sudo mkdir /data` wird ein Verzeichnis auf tiefster Ebene erstellt. Mit `sudo mount /dev/sda1 /data` wird die Festplatte temporär eingebunden. Mit `df -h` kann geprüft werden, ob der mount erfolgreich war.
	4. Um die Festplatte beim Systemstart automatisch einzubinden, muss dies in der Datei `/etc/fstab` hinterlegt werden. Zunächst wird mit `lsblk -o NAME,UUID` die Festplatten UUID ermittelt. Anschliessend wird die Datei mit `sudo nano /etc/fstab` geöffnet und durch die folgende Zeile ergänzt. Der Editor wird mit STRG+X und Bestätigung der Anfrage, ob die Ursprüngliche Datei überschrieben werden soll, beendet.

	```
	UUID=<<uuid>> /data ext4 defaults 0	0
	```
	
	5. Mit `sudo reboot` wird das Gerät neu gestartet. Dadurch wird die ssh-Verbindung unterbrochen. Wenn das erneute Einloggen funktioniert, ist alles gut gegangen.

4. Vorbereitende Software installieren:
	1. Vorbereitend muss eine Reihe von Konsolenanwendungen installiert werden. Dies geschieht mit dem folgenden Befehl.
	
	```bash
	sudo apt install apache2 mariadb-server libapache2-mod-php7.2 php7.2-gd php7.2-json php7.2-mysql php7.2-curl php7.2-mbstring php7.2-intl php-imagick php7.2-xml php7.2-zip
	```
	
	2. Um die Datenbank zu konfigurieren muss sich mit `sudo mysql` im Datenbankserver eingeloggt werden. Anschliessend wird eine neue Datenbank angelegt, ein User angelegt und dem User die entsprechenden Rechte für diese Datenbank gewährt. Wichtig ist das Semikolon (;) am Ende jeden Befehls. Anschliessend werden die Rechte mit `SHOW GRANTS FOR ncuser;` überprüft.
	
	```
	CREATE DATABASE ncdb;
	CREATE USER ncuser IDENTIFIED BY '<<password>>';
	GRANT ALL privileges ON ncdb.* TO ncuser;
	FLUSH PRIVILEGES;
	```

5. Nextcloud installieren: 
	1. Mit `wget https://download.nextcloud.com/server/releases/nextcloud-17.0.0.tar.bz2` [3] werden die Nextcloud-Programmdateien heruntergeladen und anschliessend mit `tar -xjvf nextcloud-17.0.0.tar.bz2` entpackt. Mit `ls` kann das entstandene Verzeichnis dargestellt werden. 
	2. Das Verzeichnis wird nun mit `sudo mv nextcloud/* /var/www/html` in das Web-Verzeichnis verschoben.
	3. Weiter müssen die Besitzerrechte an der Nextcloud-Instanz dem Web-Useraccount überschrieben werden. Dies geschieht mit `sudo chown -R www-data:www-data /var/www/html/`. Das Gleiche wird mit dem Datenverzeichnis getan `sudo chown -R www-data:www-data /data/`
	4. Dem Webserver muss gesagt werden, an welcher Stelle das Nextcloud installiert wurde. Dazu wird die Konfigurationsdatei unter `/etc/apache2/sites-enabled/000-default.conf` wie folgend angepasst. Dazu wird der Befehl `sudo nano <<filename>>` genutzt.
	```xml
	<VirtualHost *:80>
	    DocumentRoot /var/www/html/
	    ServerName  your.server.com
	    ServerAdmin your@email.address

	    <Directory "/var/www/html/">
		Require all granted
		AllowOverride All
		Options FollowSymLinks MultiViews

		    <IfModule mod_dav.c>
		    Dav off
		</IfModule>

	    </Directory>
	</VirtualHost>
	```
	5. Weiter müssen einige Webserver module installiert werden, und der Webserver anschliessend mit `sudo service apache2 restart` neu gestartet werden.
	```bash
	sudo a2enmod rewrite
	sudo a2enmod headers
	sudo a2enmod env
	sudo a2enmod dir
	sudo a2enmod mime
	sudo a2enmod ssl
	```
	6. Im Browser wird die IP Adresse, oder (nachdem im Router definiert) der Gerätename aufgerufen um die Konfiguration durchzuführen. Für den Adminuser `admin` und ein sicheres Passwort verwenden (Empfehlung: Zufallskombination + Klebezettel auf dem Gerät). Dieser Account wird verwendet um die Personenberechtigungen zu steuern. Für den Datenspeicherort `/data` angeben, für die Datenbankverbindung die oben festgelegten Werte verwenden.
	7. (Das Gerät kommt an dieser Stelle in eine Schleife, bei der es sich dauerhaft versucht einzuloggen. Das ist zwar nicht normal, wird aber im nächsten Schritt behoben)
	8. Mit `sudo nano /var/www/nextcloud/config/config.php` öffnen wir die config-Datei zum bearbeiten und fügen den hostnamen dem Array von trusted Domains hinzu, so dass dieser Abschnitt später wie folgt aussieht. Speichern und beenden mit STRG+X
	```php
	'trusted_domains' =>
	array (
		0 => '192.168.188.XX',
		1 => '<<hostname>>',
	),
	```
	9. Anschliessend kann die Nextcloud über `http://<<hostname>>` aufgerufen werden.

6. Nextcloud konfigurieren:
	
### Schritte, die zuhause unternommen werden müssen (viel variation möglich, daher nur oberflächlich beschrieben)

1. Gerät am Router einstecken und anschalten.
2. Im Router:
	1. In den Router einloggen
	2. IP Adresse des ODROID ermitteln.
	3. Im Router DynDNS eintrag hinterlegen.
	4. Im Router Port forwarding für Port 80 und 443 aktivieren.

3. Am ODROID:
	1. Über die Console im Gerät einloggen mit `ssh <<username>>@<<ipaddress>>`
	2. Mit `sudo nano /var/www/nextcloud/config/config.php` öffnen wir die config-Datei zum bearbeiten und fügen den hostnamen dem Array von trusted Domains hinzu, so dass dieser Abschnitt später wie folgt aussieht. Speichern und beenden mit STRG+X
	```php
	'trusted_domains' =>
	array (
		0 => '<<neue ipaddress>>',
		1 => '<<hostname>>',
		2 => 'dyndns url',
	),
	```
	3. Für die SSL Transportverschlüsselung wird der Letsencrypt [4] Certbot [5] verwendet. Dieser wird mit den folgenden Befehlen installiert. Anschliessend wird der Certbot mit `sudo certbot --apache` gestartet. WICHTIG: sowohl das Port-Forwarding, als auch der DynDNS Eintrag müssen eingetragen sein!!
	```bash
	sudo apt update
	sudo apt install software-properties-common
	sudo add-apt-repository universe
	sudo add-apt-repository ppa:certbot/certbot
	sudo apt update
	sudo apt install certbot python-certbot-apache 
	```
3. Die Nextcloud ist jetzt unter der DynDNS URL über das Internet erreichbar

### Howto:

##### User anlegen:
1. Um einen neuen Nutzer anzulegen muss sich mit dem Admin User an der Weboberfläche angemeldet werden.
2. Über das (A) am rechten oberen Rand wird das Menü aufgerufen und von dort Users/Benutzer ausgewählt. 
3. Anschliessend kann ein neuer Nutzer angelegt werden. Benutzername meint hierbei den Anzeigenamen, Email Adrresse braucht nur bei konfiguriertem Email-Server ausgefüllt werden. Gruppen kann leer bleiben. Kontingent unbegrenzt.

##### Passwort zurücksetzen:
1. Wenn kein Email-Server hinterlegt ist, kann auch kein Passwort Reset Mail versendet werden.
2. Um das Passwort zu ändern muss sich mit dem Admin User an der Weboberfläche angemeldet werden.
3. Über das (A) am rechten oberen Rand wird das Menü aufgerufen und von dort Users/Benutzer ausgewählt.
4. Nun das Feld 'neues Passwort' mit dem neuen Wunschpasswort belegen.

#### System aktualisieren:
1. Von Zeit zu Zeit sollte das Systme aktualisiert werden (jeden Monat)
2. Dazu muss sich in das Gerät eingeloggt werden, und mit `sudo apt update` die Softwareliste aktualisiert werden. Anschliessend kann mit `sudo apt upgrade` die aktualisierte Software heruntergeladen werden.

#### Links:
[1] https://wiki.odroid.com/odroid-xu4/getting_started/os_installation_guide#installation

[2] https://docs.nextcloud.com/server/17/admin_manual/installation/source_installation.html

[3] https://nextcloud.com/install/#instructions-server

[4] https://letsencrypt.org/

[5] https://certbot.eff.org/lets-encrypt/ubuntubionic-apache
