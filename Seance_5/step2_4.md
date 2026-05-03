Exercice : Step 1/4 Botnet

Mot de passe à trouve : "telnet"

Méthode de résolution : 

Objectif : `Give the name of the protocol that is attacked by the malware. This is the infection vector.`

Pour trouver le nom du protocole attaqué par le malware, nous allons analyser les processus 7295 et 7296 pour trouver des informations sur les connexions réseau établies par ces processus, et c'est à partir de ces informations que nous allons pouvoir trouver le nom du protocole attaqué par le malware.

### Analyse des connexions réseau établies par les processus 7295 et 7296

Pour analyser les connexions réseau établies par les processus 7295 et 7296, nous allons utiliser le plugin `linux.sockstat.SockStat` qui permet de trouver les différentes connexions réseau établies par les processus, et de les analyser pour trouver des informations sur les protocoles utilisés.


### Connexions réseau établies par le processus 7295

```bash
python3 vol.py -f ./../memoire/ram.img linux.sockstat --pid 7295
``` 

résultat de la commande :
```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ python3 vol.py -f ./../memoire/ram.img linux.sockstat --pid 7295
Volatility 3 Framework 2.28.1
Progress:  100.00		Stacking attempts finished            
NetNS	Process Name	PID	TID	FD	Sock Offset	Family	Type	Proto	Source Addr	Source Port	Destination Addr	Destination Port	State	Filter

4026531840	/bin/systemd	7295	7295	0	0x8938c945ec00	AF_INET	STREAM	TCP	192.168.0.2	59616	192.168.0.1	56999	ESTABLISHED	-
4026531840	/bin/systemd	7295	7295	3	0x8938c945f500	AF_INET	STREAM	TCP	0.0.0.0	1230	0.0.0.0	0	LISTEN	-
```

C'est une connexion TCP établie entre l'adresse IP, après avoir établi la connexion, cela ne va pa trop nous aider à trouver le nom du protocole attaqué par le malware.

### Cherchons les commandes lancer dans le shell durant la session

```bash
python3 vol.py -f ./../memoire/ram.img linux.bash
``` 

lien vers le résultat de la commande : [bash.txt](./extrait_bash.txt)

Extrait de la liste des commandes lancées dans le shell durant la session qui nous intéresse :
```
5407	bash	2023-12-01 15:50:57.000000 UTC	sudo apt update
5407	bash	2023-12-01 15:50:57.000000 UTC	sudo apt upgrade
5407	bash	2023-12-01 15:50:57.000000 UTC	sudo apt install warmux
5407	bash	2023-12-01 15:50:57.000000 UTC	telnet 134.206.250.78 23
5407	bash	2023-12-01 15:50:57.000000 UTC	nc 134.206.250.78 23
5407	bash	2023-12-01 15:50:57.000000 UTC	ip a
5407	bash	2023-12-01 15:50:57.000000 UTC	ping google.com
5407	bash	2023-12-01 15:50:57.000000 UTC	ping 8.8.8.8
5407	bash	2023-12-01 15:50:57.000000 UTC	ping 8.8.8.8
5407	bash	2023-12-01 15:50:57.000000 UTC	ping -c 1 8.8.8.8
5407	bash	2023-12-01 15:50:57.000000 UTC	ping 8.8.8.8
5407	bash	2023-12-01 15:50:57.000000 UTC	sudo VBoxLinuxAdditions.run 
5407	bash	2023-12-01 15:50:57.000000 UTC	telnet 9.9.9.9 23
5407	bash	2023-12-01 15:50:57.000000 UTC	ip a 192.168.0.1
5407	bash	2023-12-01 15:50:57.000000 UTC	telnet 192.168.0.1 2223
5407	bash	2023-12-01 15:50:57.000000 UTC	telnet 192.168.0.1
5407	bash	2023-12-01 15:50:57.000000 UTC	ip r
5407	bash	2023-12-01 15:50:57.000000 UTC	sudo ./VBoxLinuxAdditions.run 
5407	bash	2023-12-01 15:50:57.000000 UTC	ip a
5407	bash	2023-12-01 15:50:57.000000 UTC	uname -a
5407	bash	2023-12-01 15:50:57.000000 UTC	 
5407	bash	2023-12-01 15:50:57.000000 UTC	telnet 192.168.0.1 23
5407	bash	2023-12-01 15:50:57.000000 UTC	dig google.com
5407	bash	2023-12-01 15:50:57.000000 UTC	telnet 134.206.250.78 23
5407	bash	2023-12-01 15:50:57.000000 UTC	sudo apt update
5407	bash	2023-12-01 15:50:57.000000 UTC	dig google.com
5407	bash	2023-12-01 15:50:57.000000 UTC	reboot
5407	bash	2023-12-01 15:50:57.000000 UTC	ip a
5407	bash	2023-12-01 15:50:57.000000 UTC	wget google.com
5407	bash	2023-12-01 15:50:57.000000 UTC	nc 192.168.0.1 23
5407	bash	2023-12-01 15:50:57.000000 UTC	./VBoxLinuxAdditions.run 
5407	bash	2023-12-01 15:50:59.000000 UTC	cd/
5407	bash	2023-12-01 15:51:00.000000 UTC	cd /
5407	bash	2023-12-01 15:51:06.000000 UTC	sudo su
5445	bash	2023-12-01 15:51:28.000000 UTC	chmod +x /home/host/Downloads/warmux 
5445	bash	2023-12-01 15:53:50.000000 UTC	0
5445	bash	2023-12-01 15:53:50.000000 UTC	OLDPWD=/
5445	bash	2023-12-01 15:53:50.000000 UTC	r-Xwaylandauth.G1Y4E2
5445	bash	2023-12-01 15:53:50.000000 UTC	cd /home/host/Downloads/
5445	bash	2023-12-01 18:53:58.000000 UTC	SUDO_USER=host
5445	bash	2023-12-01 18:53:58.000000 UTC	ping 8.8.8.8
5445	bash	2023-12-01 18:54:03.000000 UTC	./warmux 
```

Dans cette liste de commandes lancées dans le shell durant la session, on peut voir que la commande `telnet` a été utilisée pour établir une connexion avec l'adresse IP 134.206.250.78.
