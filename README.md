# Baue deine eigene Cloud

### Ablauf:
Zeit | Programmpunkt
-- | --
17:00 | Was ist das für ein Gerät? Was ist Nextcloud?
17:30 | Das Gerät vorbereiten & das Netzwerk einrichten
18:30 | Nextcloud installieren
20:00 | Pizza und Theorie: DynDNS und Port Forwarding
21:00 | Ende (Eventuell wirds auch 22:00)

### Inhalt:
Odroid HC4, Stromkabel + Anschluss EU, LAN Kabel, SD Card, Festplatte

### Lämpchen und deren Bedeutung:
* Rotes Lämpchen (PWR): Das Gerät hat Strom
* Blaues Lämpchen, blinkend (ALIVE): Das Gerät ist in Betrieb.
* Grünes Lämpchen, leuchtend: Auf die Festplatte wird zugegriffen.

### Benötigte Software auf dem Computer:
* Windows: Putty ist ein Bash Terminal, mit welchem wir uns mit dem Gerät verbinden. Alternativ kann unter Windows 10 auch der Linux Kernel verwendet werden
* Mac/Ubuntu: Terminal
* WLAN `techlabwlan`. Passwort `12341234`

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
0. Gerät in Betrieb nehmen: Bereits mit einer UBUNTU Installation vorbereitet ist die SD Karte. Soll das System neu installiert werden, kann die UBUNTU Installation unter [[1]](https://wiki.odroid.com/odroid-hc4/getting_started/os_installation_guide) bezogen werden. Die ISO Datei muss dazu auf den USB Stick “gebrannt” werden. Unter Windows geschieht das mit dem Tool  [_RUFUS_ [2]](https://rufus.ie/en/)

1. Netzwerk verwalten: Beim Verbinden des Geräts mit dem Router wird automatisch eine Netzwerkaddresse zugewiesen. Diese kann zum Beispiel über das Webinterface des Routers herausgefunden werden. Mit einem angeschlossenen Bildschirm wäre dies mit dem Kommandozeilenbefehl `ip addr` möglich. Der Laptop muss mit dem WLAN `techlabwlan` verbunden sein.

2. Gerät Konfigurieren: 
	1. Auf das Gerät wird mittels `ssh` zugegriffen. Das ist möglich mit dem Programm [Putty (Download [3])](https://www.chiark.greenend.org.uk/~sgtatham/putty/), über den Windows 10 Linux Subsystem [[4]](https://docs.microsoft.com/de-ch/windows/wsl/install), oder über das Terminal bei Mac und Ubuntu. Der Befehl zum verbinden lautet `ssh root@<<ip-addresse>>`. Die IP wurde in Schritt 1 ermittelt. Das Passwort zu diesem Zeitpunkt lautet `odroid`.
	2. Als erstes sollte das System upgedated werden `apt update` lädt dabei die aktuellsten Software Listen herunter. Anschliessend wird `apt upgrade` ausgeführt um das System zu aktualisieren.
	3. Um das System vor ungewünschtem Eindringen zu schützen muss das root geändert werden. Mit dem Befehl `passwd` wird die Änderung des Passworts des eingeloggten Users eingeleitet. Das Passwort sollte möglichst hart sein, und wird nur selten verwendet. (Empfehlung: Zufallskombination + Klebezettel auf dem Gerät). 
	4. Zur sichereren Bedienung wird ein weiterer Benutzer angelegt. Dies geschieht mit dem Befehl `adduser <<username>>`. Dieser Nutzer wird unser Standartnutzer sein. Bei Linux/Unix Systemen wird empfohlen auf verschiedenen Geräten den gleichen Nutzernamen zu wählen. Mit `usermod -aG sudo <<username>>` wird der Nutzer der sudo Gruppe hinzugefügt, womit die Ausführung mit privilegierten Rechten möglich wird. Zusätzlich müssen wir 
	5. Um das Gerät (in unserem Setting) eindeutig identifizierbar zu machen wird mit `hostnamectl set-hostname <<name>>` der Gerätename geändert.
	6. Die Verbindung mit dem Gerät wird mit `exit` getrennt. Anschliessend verbinden wir uns wieder mit unserem neu angelegten Nutzer: `ssh <<username>>@<<ip-address>>`

3. Festplatte bereitstellen:
	1. Zunächst wird mit `sudo fdisk -l` die Festplatten-ID ermittelt. Anschliessend wird das Formatierungstool aufgerufen mit `sudo fdisk <<ID>>` (beim odroid ist ID immer `/dev/sda`). Die Hilfe zum Tool kann mit `m` (menu) ausgegeben werden. Um die Festplatte zu formatieren wählen wir `n` (new partition) aus. Im Dialog legen wir den primary Partition Type fest (`p`) und bestätigen die Standardwerte der restlichen Eingaben. Mit `w` (write) werden die Änderungen übernommen. Anschliessend kann die Partition mit `sudo fdisk -l` geprüft werden.
	2. nach der Partitionierung muss anschliessend das Filesystem bereitgestellt werden. Für das EXT4 Filesystem passiert das mit dem Befehl `sudo mkfs.ext4 /dev/sda1` (ID _sda1_ vorhergehend ermittelt).
	3. Anschliessend wird die Festplatte in den Verzeichnisbaum eingebunden. Mit `sudo mkdir /data` wird ein Verzeichnis auf tiefster Ebene erstellt. Mit `sudo mount /dev/sda1 /data` wird die Festplatte temporär eingebunden. Mit `df -h` kann geprüft werden, ob der mount erfolgreich war.
	4. Um die Festplatte beim Systemstart automatisch einzubinden, muss dies in der Datei `/etc/fstab` hinterlegt werden. Zunächst wird mit `lsblk -o NAME,UUID` die Festplatten UUID ermittelt. Anschliessend wird die Datei mit `sudo nano /etc/fstab` geöffnet und durch die folgende Zeile ergänzt. Der Editor wird mit STRG+X und Bestätigung der Anfrage, ob die Ursprüngliche Datei überschrieben werden soll, beendet.

	```
	UUID=<<uuid>> /data ext4 defaults 0	0
	```
	
	5. Mit `sudo reboot` wird das Gerät neu gestartet. Dadurch wird die ssh-Verbindung unterbrochen. Wenn das erneute Einloggen funktioniert, ist alles gut gegangen.

4. Vorbereitende Software installieren:
	1. Vorbereitend muss eine Reihe von Konsolenanwendungen installiert werden. Dies geschieht mit dem folgenden Befehl.
	
	```bash
	sudo apt install apache2 mariadb-server libapache2-mod-php7.4 php7.4-gd php7.4-mysql php7.4-curl php7.4-mbstring php7.4-intl php7.4-gmp php7.4-bcmath php-imagick php7.4-xml php7.4-zip
	```
	
	2. Um die Datenbank zu konfigurieren muss sich mit `sudo mysql` im Datenbankserver eingeloggt werden. Anschliessend wird eine neue Datenbank angelegt, ein User angelegt und dem User die entsprechenden Rechte für diese Datenbank gewährt. Wichtig ist das Semikolon (;) am Ende jeden Befehls. Anschliessend werden die Rechte mit `SHOW GRANTS FOR ncuser;` überprüft.
	
	```
	CREATE DATABASE ncdb;
	CREATE USER ncuser IDENTIFIED BY '<<password>>';
	GRANT ALL privileges ON ncdb.* TO ncuser;
	FLUSH PRIVILEGES;
	```

5. Nextcloud installieren: 
	1. Mit `wget https://download.nextcloud.com/server/releases/nextcloud-22.2.3.tar.bz2` [5] werden die Nextcloud-Programmdateien heruntergeladen und anschliessend mit `tar -xjvf nextcloud-22.2.3.tar.bz2` entpackt. Mit `ls` kann das entstandene Verzeichnis dargestellt werden. 
	2. Das Verzeichnis wird nun mit `sudo mv nextcloud/* /var/www/html` in das Web-Verzeichnis verschoben.
	3. Weiter müssen die Besitzerrechte an der Nextcloud-Instanz dem Web-Useraccount überschrieben werden. Dies geschieht mit `sudo chown -R www-data:www-data /var/www/html/`. Das Gleiche wird mit dem Datenverzeichnis getan `sudo chown -R www-data:www-data /data/`
	4. Dem Webserver muss gesagt werden, an welcher Stelle das Nextcloud installiert wurde. Dazu wird eine neue Konfigurationsdatei unter `/etc/apache2/sites-enabled/nextcloud.conf` angelegt. Der Befehl dazu lautet`sudo nano <<filename>>`.
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
	5. Die Konfiguration wird anschliessend mit dem Befehl `a2ensite nextcloud.conf` aktiviert.
	6. Weiter müssen einige Webserver module installiert werden, und der Webserver anschliessend mit `sudo service apache2 restart` neu gestartet werden.
	```bash
	sudo a2enmod rewrite
	sudo a2enmod headers
	sudo a2enmod env
	sudo a2enmod dir
	sudo a2enmod mime
	sudo a2enmod ssl
	```
	7. Im Browser wird die IP Adresse, oder (nachdem im Router definiert) der Gerätename aufgerufen um die Konfiguration durchzuführen. Für den Adminuser `admin` und ein sicheres Passwort verwenden (Empfehlung: Zufallskombination + Klebezettel auf dem Gerät). Dieser Account wird verwendet um die Personenberechtigungen zu steuern. Für den Datenspeicherort `/data` angeben, für die Datenbankverbindung die oben festgelegten Werte verwenden.
	8. Auf der Kommandozeile mit `sudo nano /var/www/nextcloud/config/config.php` öffnen wir die config-Datei zum bearbeiten und fügen den hostnamen dem Array von trusted Domains hinzu, so dass dieser Abschnitt später wie folgt aussieht. Speichern und beenden mit STRG+X
	```php
	'trusted_domains' =>
	array (
		0 => '192.168.188.XX',
		1 => '<<hostname>>',
	),
	```
	10. Anschliessend kann die Nextcloud über `http://<<hostname>>` aufgerufen werden.

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
	2. Mit `sudo nano /var/www/html/config/config.php` öffnen wir die config-Datei zum bearbeiten und fügen den hostnamen dem Array von trusted Domains hinzu, so dass dieser Abschnitt später wie folgt aussieht. Speichern und beenden mit STRG+X
	```php
	'trusted_domains' =>
	array (
		0 => '<<neue ipaddress>>',
		1 => '<<hostname>>',
		2 => '<<dyndns url>>',
	),
	```
	3. Für die SSL Transportverschlüsselung wird [Letsencrypt [7]](https://letsencrypt.org/) und das [acme.sh [8]](https://github.com/acmesh-official/acme.sh) script verwendet. Dieses wird mit den folgenden Befehlen installiert. Die Email-Adresse wird verwendet, um vor einem ablaufenden Zertifikat zu warnen.
	```bash
	wget -O -  https://get.acme.sh | sh -s email=<<my@example.com>>
	mkdir /var/www/html/.well-known
	chmod -R g+w /var/www/html/.well-known
	
	~/.acme/acme.sh --issue -d <<dyndns doman>> -w /var/www/html
	
	mkdir -p /.ssl/<<dyndns url>>
	chmod -R g+w /.ssl/<<dyndns url>>
	touch /.ssl/<<dyndns url>>/cert.pem
	touch /.ssl/<<dyndns url>>/key.pem
	touch /.ssl/<<dyndns url>>/fullchain.pem
	
	~/.acme/acme.sh --install-cert -d <<dyndns url>> --cert-file /.ssl/<<dyndns url>>/cert.pem --key-file /.ssl/<<dyndns url>>/key.pem --fullchain-file /.ssl/<<dyndns url>>/fullchain.pem --reloadcmd "service apache2 force-reload"
	
	a2enmod ssl
	a2ensite default-ssl
	
	```
	
	4. Anpassen der nextcloud.config mit `sudo nano /etc/apache2/sites-enabled/nexctloud.config`
	```bash
	<VirtualHost *:80>
	  ServerName <<dyndns url>>.ch
	  Redirect "/" "https://<<dyndns url>>.ch/"
	</VirtualHost>
	
	<VirtualHost *:443>
	  DocumentRoot /var/www/html/
	  ServerName  <<dyndns url>>.ch
	  
	  SSLEngine on
	  SSLCertificateFile /.ssl/<<dyndns url>>/cert.pem
	  SSLCertificateKeyFile /.ssl/<<dyndns url>>/key.pem
	  SSLCertificateChainFile /.ssl/<<dyndns url>>/fullchain.pem
	  
	  <Directory /var/www/html/>
	    Require all granted
	    AllowOverride All
	    Options FollowSymLinks MultiViews
	    
	    <IfModule mod_dav.c>
	      Dav off
	    </IfModule>
	  </Directory>
	</VirtualHost>
	```
	5. Apache neu starten via `sudo service apache2 restart`
	
4. Die Nextcloud ist jetzt unter der DynDNS URL über das Internet erreichbar

### Howto:

##### User anlegen:
1. Um einen neuen Nutzer anzulegen muss sich mit dem Admin User an der Weboberfläche angemeldet werden.
2. Über das (A) am rechten oberen Rand wird das Menü aufgerufen und vonls - dort Users/Benutzer ausgewählt. 
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
[1] https://wiki.odroid.com/odroid-hc4/getting_started/os_installation_guide

[2] https://rufus.ie/en/

[3] https://www.chiark.greenend.org.uk/~sgtatham/putty/

[4] https://docs.microsoft.com/de-ch/windows/wsl/install

[5] https://docs.nextcloud.com/server/17/admin_manual/installation/source_installation.html

[6] https://nextcloud.com/install/#instructions-server

[7] https://letsencrypt.org/

[8] https://github.com/acmesh-official/acme.sh
