# LB03
## Inhaltsverzeichnis
[Einleitung](#Einleitung)
  - [Realisierung](#Realisierung)
    - [placeholder](#placeholder)
  - [Testen](#testing)

## Einleitung
Für diese LB möchte ich eine Docker-Compose erstellen, welche eine MySQL-Datenbank per MyPHP ins LAN verfügbar stellt. Als Grundlage für Docker nehme ich eine Kopie der VM, welche Herr Berger in seinem M300 Github zur Verfügung stellte.

### Benötigte Vagrant-System-Änderungen
Um die Komposition besser testen zu können, nehme ich die VM in mein LAN auf, was bedeutet, dass es die IP-Adresse *192.168.0.45 /24* bekommt. Vor der Abgabe werde ich dies wieder auf die verlangte Adresse *192.168.60.101 /24* ändern.
<br>

Ebenso füge ich diese Provision ins Vagrantfile ein, um mein Dockerfile und Docker-Compose immer verfügbar zu haben:
```shell
  # Shell Provision
  config.vm.provision "shell", inline: <<-SHELL 
  
   apt-get update
   apt-get install -y docker-compose

   mkdir compose-projekt
   mkir compose-projekt/src
   sudo wget -P ./compose-projekt/src https://raw.githubusercontent.com/Ben1702/M300_LB_Stutz/main/lb3/docker-compose.yml
   
  SHELL
```

## Realisierung

### placeholder

<br>

## Testen

<br>

## Quellenangaben

