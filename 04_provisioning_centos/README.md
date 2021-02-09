# Foreman Training - Teil 4 - Provisionieren CentOS

Das Provisionieren besteht aus:
  - Betriebssystem und -version
  - OS Architektur
  - Installations Medium (Repository Server)
  - Provisionierungstemplates

Zusaetlich lassen wir mit Foreman die Vergabe von IP Adressen (DHCP) und die Namensaufloesung (DNS) verwalten.

## Domain

Foreman Login -> Infrastructure -> Domains -> 'example42.training'

- Tab Domain: DNS domain: 'example42.training'
- Tab Domain: DNS Proxy: 'foreman.example42.training'

Submit

## Netzwerk

Foreman Login -> Infrastructure -> Subnets -> Create Subnet

- Tab Subnet: Name: example42.training
- Tab Subnet: Network Address: 10.100.10.0
- Tab Subnet: Network Prefix: 24
- Tab Subnet: Gateway Address: 10.100.10.101
- Tab Subnet: Primary DNS Server: 10.100.10.101
- Tab Subnet: IPAM: DHCP
- Tab Subnet: Start of IP Range: 10.100.10.120
- Tab Subnet: End of IP Range: 10.100.10.240

- Tab Domains: example42.training ausaehlen

- Tab Proxies: foreman.example42.training bei DHCP, TFTP, HTTPBoot und Reverse DNS auswaehlen

Submit

## Provisioning OS

Es koennen meherere OS ausgewaehlt werden.
Das OS mit Version und Architektur der Foreman Instanz ist automatisch hinzugefuegt worden.

## Provisioning Repository Server

Default Repository Server fuer unterschiedliche Betriebssysteme sind bereits standardmaessig angelegt.
Hier eventuell die Lifecycle Environment Repositories auswaehlen.

## Provisioning Templates

Foreman Login -> Hosts -> Provisioning Templates

Templates unterscheiden sich nach Funktionalitaet:

  - PXELinux, PXEGrub, PXEGrub2 : werden auf den TFTP Server deployed
  - Provision : Kickstart oder Preseed fuer unattended Installation
  - Finish : Post-Install Scripts
  - user_data: Post-Install fuer Cloud VMs
  - Script : generische Skripte
  - iPXE : {g,i}PXE anstelle von PXE


Normalerweise werden nur die ersten 3 benoetigt.

Standard Templates sind "Locked". Niemals Standard Templates editieren, immer Klonen!

Beispiel Suchen: kind=PXELinux

### Provisionierungs Templates mit OS Assoziieren

Foreman Login -> Hosts -> Provisioning Templates

1. Stelle: Templates mit OS Assoziieren

1.a. kind = PXELinux

"Kickstart default PXELinux" auswaehlen -> Association -> OS auswaehlen

1.b. kind = provision

"Kickstart default" auswaehlen -> Association -> OS auswaehlen

1.c. kind = finish

"Kickstart default finish" auswaehlen -> Association -> OS auswaehlen

1.d. PXE bauen

"Build PXE Default"

### Installation media

Wenn man einen eigenen anstelle der CentOS Mirror verwenden moechte:

Foreman Login -> Hosts -> Installation Media -> Create medium

Name eintragen, bei "Path" den http Pfad eintragen.
Achtung: auf Namensaufloesung achten!

Wichtig: im Linuxhotel bitte unbedingt den Mirror verwenden: path: `http://centos.linuxhotel.de/7/os/x86_64`

Operatingsystem Family angeben!

#### lokalen Repo Mirror bekannt machen (nur wenn man das CentOS Repository als Content erzeugt hat.)

Katello Login -> Hosts -> Installation Media -> Create Installation Medium

- Name: CentOS Katello
- Path: http://katello.example42.training/pulp/repos/Default_Organization/Library/custom/CentOS7/os_x86_64/
- Operating System Family: RedHat

Submit

### OS mit Provisionierungs Templates assoziieren

Foreman Login -> Hosts -> Operating Systems

OS auswaehlen

2. Stelle: OS mit Templates Assoziieren

- Partition Table -> "Kickstart default" auswaehlen
- Installation media -> "CentOS 7 mirror" auswaehlen (oder das vorher angelegten Installation media auswaehlen)
- Templates -> alle Templates auswaehlen, die man auswaehlen kann.

Submit

## Host Group

Wenn man Puppet einsetzen will, brauchen Host Groups das Puppet Environment

Foreman Login -> Configure -> Host Groups -> Create Host Group

Tab Host Group:

- Name: Hostgruppe auswaehlen

Die folgenden Eintraege werden nur bei der Verwendung von Puppet benoetigt:

- Environment: Production
- Puppet Proxy: foreman.example42.training
- Puppet CA Proxy: foreman.example42.training

Tab Puppet Classes

keine Klasse auswählen !!

Tab Network

- Domain: example42.training
- IPv4 Subnet: example42.training

Tab Operating System

- Architecture: x86_64
- Operatingsystem: CentOS...
- Media: CentOS Mirror (oder den vorher angelegten auswählen)
- Partition Table: Kickstart
- PXE Loader: PXELinux BIOS
- Root Password: <eines setzen>

Wenn Puppet explizit ausgeschaltet werden soll:

Tab Parameters:

Host Group Parameters: Add Parameter:

enable-puppet: boolean: false

Wenn Puppet 6 aktiviert werden soll:

enable-puppetlabs-puppet6-repo: boolean: true

Submit

## Host erzeugen in VirtualBox:

Hinweis: die Images werden initial in eine RAM Disk geladen. Deshalb benoetigt die VM mindesten 1516 MB RAM.

New -> Host -> 2048 MB RAM -> 8 GB HDD

Boot Einstellungen aendern: 1. Festplatte -> 2. Netzwerk

Netzwerk aendern: gleiches vboxnet, wie foreman.example42.training

MAC Adresse notieren

## Host in Foreman anlegen

Foreman Login -> Hosts -> Create Host

ACHTUNG: nicht docker als hostname nehmen. Dieser Name wird in Teil 4 verwendet.

Host Tab:
- Name: hostname (ohne Domain)
- Hostgroup: Training

Die folgende Einstellung ist nur vorhanden, wenn man in der Hostgruppe Puppet aktiviert hat.

- Environment: Production (sollte automatisch aus der Hostgroup rausfallen)

Interfaces Tab:
- Edit
  - Mac Adresse eintragen und OK zum speichern

ACHTUNG: MAC Adresse mit Doppelpunkten eintragen!!! aa:bb:cc:dd:ee

Operating System Tab:

- Operating System: auswaehlen (CentOS Linux 7.3...)
- Media: Mirror waehlen
- Partition Table: Kickstart default
- PXE Loader: PXELinux BIOS

Submit

Es erscheint ein "New in progress" Balken.

## Host in VirtualBox starten

Achtung: bitte mit dem Starten der VM etwas warten. (ca 5 min!!!!!!!)

Foreman muss im Hintergrund die Images für den TFTP Server herunterladen!!!
Das kann mit VirtualBox einige Zeit dauern....

Wenn man alles richtig gemacht hat, bootet die VM via DHCP und fängt die Installation an.
Wenn man timeout oder not found Fehler bekommt, hat man nicht lange genug gewartet.

Die Installation kann je nach verwendetem Repository Server einige Zeit dauern.

Zur Überprüfung der Installation kann man dann von der Foreman Instanz aus via SSH auf den neu erzeugten Host zugreifen:

    ssh <name> -l root
    yum install -y net-tools
    netstat -rn
    cat /etc/resolv.conf
    ping -c1 heise.de


Weiter geht es mit Teil 5 [Provisionieren von Debian](../05_provisionining_debian)
Oder mit Teil 6 [Deprovisionieren](../06_deprovision)