pas_logging
===============

Diese Anwendunng stellt in Zusammenhang mit RabbitMQ und einer PostgreSQL Datenbank die Basisstruktur für das zukünftige Logging meiner Anwendungen zur Verfügung. Dadurch wird dann auch ZMQ beim lokalen Logging in allen Anwendungen abgelöst.

Die Anwendung arbeitet unter .Net Core 8 und ist in C# geschrieben worden. Sie arbeitet unter Windows und Linux.

Die Anwendung hat mehrere Funktionalitäten.

## Datenbankverbindung testen

Über den Befehl "dbcheck" kann das Programm einen Kontaktversuch mit der Datenbank durchführen.

./pas_psql_logger dbcheck --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 


## Datenbankstrukturen löschen

Über den Befehl "dbdrop" kann das Programm die notwendigen Strukturen in einer erreichbaren Datenbank löschen. 

./pas_psql_logger dbdrop --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 


## Datenbankstrukturen anlegen

Über den Befehl "dbcreate" kann das Programm die notwendigen Strukturen in einer erreichbaren Datenbank anlegen. Zur Speicherung wird eine partitionierte Tabelle mit vielen Unterpartitionen genutzt. Die Partitionierung erfolgt mit 4 Tabellen pro Monat. Dabei bestimmt der Parameter "months", wieviele Monate angelegt werden sollen. Dabei zählt der aktuelle Monat immer mit. Bei months = 3 wird also aktuelle Monate und zwei zusätzliche Monate angelegt

./pas_psql_logger dbcreate --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 --months 3


## Weitere notwendige Partitionen anlegen

Über den Befehl "dbmaint" kann das Programm weitere Partitionierungen anlegen. Dazu muß man die Option --months definieren mit der Anzahl der Monate, für Tabellen angelegt werden sollen. Der aktuelle Monat zählt dabei immer mit.  Bei z.B. einer 2 werden die Tabellen für den aktuellen Monate und dem nächsten Monat angelegt.

./pas_psql_logger dbmaint --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 --months=2


## Alte Partitionen entfernen

Über den Befehl "dbgc" kann das Programm nicht mehr notwendige Partitionen löschen. Dabei lässt das System die Monate unangetastet, die über die Variable months definiert werden. Hat dieser Parameter den Wert 2, dann wird der aktuelle Monat und ein Monat davor nicht verändert.

./pas_psql_logger dbgc --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 --months=2


## Testdaten - Test LOGs direkt in die Datenbank schreiben

Über den Befehl "dbmsg" kann das Programm Testnachrichten in die Datenbnak schreiben. Dabei können die so erzeugten Nachrichten über einen Zeitraum verteilt werden. Diese Testdaten sollen dazu genutzt werde, um Oberflächen in Superset definieren zu können.

./pas_psql_logger dbmsg --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 --months=2 --count=100000 --tsize=100


## Testdaten - Test CATI LOGs direkt in die Datenbank schreiben

Über den Befehl "dbcatimsg" kann das Programm spezielle LOG-Meldungen in die Datenbank schreiben. Der Erfolgscode ist dabei "501".

./pas_psql_logger dbcatimsg --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 --months=2 --count=100000 --tsize=100


## Testdaten - Test LOGs nach RabbitMQ senden

Über den Befehl "mqmsg" kann das Programm Testnachrichten an RabbitMQ senden. Dabei können die so erzeugten Nachrichten über einen Zeitraum verteilt werden. Diese Testdaten sollen dazu genutzt werden, um Oberflächen in Superset definieren zu können. Dieser Befehl kann in Testumgebungen mit einer anderen Instanz des Programmes arebiten, dass den Befehl dbserver ausführt.

./pas_psql_logger mqmsg --mqserver=127.0.0.1 -mqexchange=amq.topic --mqpassword=... --count=100000 --months=2 


## RabbitMQ-Strukturen - Legt die gewünschte Exchange, Queue und verbindet diese mittels des eingestellten RoutingKey

Über den Befehl "mqcreate" kann das Programm Basisstrukturen auf dem RabbitMQ Server anlegen. 

./pas_psql_logger mqcreate --mqserver=127.0.0.1 -mqexchange=amq.topic --mqpassword=... --mquser=mquser --mqqueue=logqueue --mqvhost=/ --mqrkey=evlog --verbose


## Serverdienst

Über den Befehl "dbservice" wird das Programm im Serverdienst gestartet. Es verbindet sich mit RabbitMQ und einer relationalen Datenbank und verschiebt die LOG-Meldungen in die Datenbank.´
Folgende Optionen müssen dabei gesetzt sein:Das Programm benötigt dabei die folgenden Informationen (neben den Datenbakinformationen s.o.) 

./pas_psql_logger dbservice --mqserver=nextgeneration.gessgroup.de --mquser=... --mqpassword=... --mqqueue=edb_log --dbserver=127.0.0.1 -dbname=paslog --dbpassword=... --dbport=5432 --tsize=100 --verbose --debug


- mqserver: Adresse des RabbitMQ Servers
- mquser: Benutzername im RabbitMQ Server
- mqpassword: Passwort beim Einloggen in den RabbitMQ Servers
- mqqueue: In welcher Queue werden die LOG-Daten erwartet
- dbserver: Adresse des Datenbankservers
- dbname: Name der Datenbank, in die LOGs abgelegt werden
- dbpassword: Kennwort beim Zugriff auf die Datenbank
- dbport: Port des Datenbankservers
- tsize: Transaktionsgröße beim Schreiben der Daten in die Datenbank
- debug: Alle Nachrichten werden mit debug=true geloggt
- verbose: Zusätzliche Informationen werden auf der Konsole ausgegeben

### Default Settings

Wenn das Programm lokal eine Datei "settings.txt" findet, dann liest es diese Datei (zeilenweise)) und füllt die folgenden Werte (in der Reihenfolge - bei einer Leerzeile wird der Eintrag übersprungen):

- mqserver
- mquser
- mqpassword
- mqqueue
- mqexchange
- mqroutingkey
- mqvhost
- dbserver
- dbname
- dbuser
- dbpassword
- dbport
- tsize
- minForwardLevel
- forwardRoutingKey


## Installation

Für die Nutzung braucht man einen Account zu einer relationalen angelegten Datenbank (Also: login, password und db name). Weiterhin einen RabbitMQ-Account (host, vhost, login, password). Damit kommt man schon recht gut weiter. Es gibt Befehle, um eine PSQGL-Anbindung zu testen, Befehle um die notwendigen DB-Strukturen und RabbitMQ-Strukturen anzulegen. Das System kann komplett im debug-Modus gefahren werden.

Es gibt keine Docker-Dateien - der Aufwand ist viel zu groß und ein .Net Core 8.0 zu installieren ist auf jedem Rechner ziemlich einfach.
