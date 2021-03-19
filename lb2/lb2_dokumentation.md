# M300 LB2 
## Vagrant-VM initieren
Ein Vagrant-Ordner erstellen und dort die Vagrantfile-Vorlage erstellen.
```bash
mkdir Vagrant
cd vagrant
vagrant init
```

Nun kann man das Vagrantfile, am besten mit Visual Studio, bearbeiten. <br>
Ich fing mit der sogenannten "Box" an. Dies ist eigentlich nichts anderes als die OS, welche die VM bei Start bekommt. <br>
Ich habe mich für Debian Jessie entschieden. Alle Boxen können unter https://app.vagrantup.com/boxes/search recherchiert werden.<br>
Innerhalb Vagrantfile:
```bash
Vagrant.configure("2") do |config|

  config.vm.box = "debian/jessie64"

```
Wenn man nun testweise die VM (mit vagrant up) aufstartet, sollte es als Debian starten.<br>
![Erstes Vagrant up](Dokumentation-data/first_vagrant_up.png)

Nach diesem Test am besten die VM wieder "herunterfahren" mit Vagrant destroy und danach im Vagrantfile folgende Zeilen angeben:
```bash
   config.vm.network "public_network", ip: "192.168.0.45", bridge: "Intel(R) I211 Gigabit Network Connection"
   config.vm.hostname  = "Benjamin-Debian-Sambaclient"
```
Damit setze ich fest dass die Maschine auf mein lokales Netzwerk zugreifen soll, welche IP-Adresse und welches Interface es dafür nehmen soll. Dazu kommt dann noch der Hostname, welcher im LAN verwendet werden soll.<br>
<br>
Als vorerst letztes kommen noch ein paar "Hardware"-Einstellungen. Hier setze ich fest, dass die Maschine 2048 bytes an RAm und 2 CPU-Kerne verwenden darf. Auch hier setze ich noch den Namen der Maschine fest.
```bash
config.vm.provider "virtualbox" do |vb|
#   # Display the VirtualBox GUI when booting the machine
#   vb.gui = true
#
#   # Customize the amount of memory and CPUs on the VM:
   vb.memory = "2048"
   vb.cpus = "2"
#   # Determine the VM-Name:
   vb.name = "Benjamin-Debian-Sambaclient"
 end
```

---
## Samba-Funktion erstellen
