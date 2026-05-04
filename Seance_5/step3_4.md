Exercice : Step 3/4 Botnet

Mot de passe à trouve : "helanhaiya.999apk.top:56999"

Méthode de résolution : 


Objectif : `Give the domain name and the port of the command and control server (also known as C&C or C2). Syntax : name:port.`

A l'aide des résultats du step 2/4, On  peu reconstruire le scnario d'attaque, et on peut voir que le malware a établi une connexion avec l'adresse IP 134.206.250.78 et le port 56999.

Je pense qu'après avoir installer le jeu warmux, le malware a utilisé la commande `telnet` pour établir une connexion avec l'adresse IP 134.206.250.78 et le port 56999.

Maintenant notre intérêt est de trouver le nom de domaine associé à l'adresse IP 134.206.250.78.

### Trouver le nom de domaine associé à l'adresse IP

En suivant la piste donné sur le DNSChef qui est un spoofing DNS, j'ai pu remarquer que le domaine  ne peut pas etre resolu en utilisant les serveurs DNS classiques, et que le domaine est associé à l'adresse IP.
### Analyse de warmux

Essayer de rechercher le nom de domaine dans le processus warmux, de pid `6175` avec le plugin `linux.proc.Maps` pour trouver les différentes zones de mémoire utilisées par le processus, et de les analyser pour trouver des informations sur le nom de domaine associé à l'adresse IP.

Après avoir tester plusieurs alternatives vers cette hypothèse de warmux, je n'ai pas trouvé de piste exploitable pour trouver le nom de domaine associé à l'adresse IP.

J'ai tenté de faire filtrage avec `grep` pour trouver le nom de domaine dans les différentes zones de mémoire utilisées par le processus warmux, mais je n'ai pas trouvé de piste exploitable pour trouver le nom de domaine associé à l'adresse IP.

J'ai tenté aussi de voir les différentes connexions réseau établies par le processus warmux avec le plugin `linux.sockstat.SockStat` pour trouver des informations sur les protocoles utilisés, mais je n'ai pas trouvé de piste exploitable pour trouver le nom de domaine associé à l'adresse IP.

toujours rien trouvé de piste exploitable pour trouver le nom de domaine associé à l'adresse IP.




### Analyse de la mémoire  du processus /bin/systemd de pid 7295
On connait deja le port a l'aide du step 2/4

Rappel
```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ python3 vol.py -f ./../memoire/ram.img linux.sockstat --pid 7295
Volatility 3 Framework 2.28.1
Progress:  100.00		Stacking attempts finished            
NetNS	Process Name	PID	TID	FD	Sock Offset	Family	Type	Proto	Source Addr	Source Port	Destination Addr	Destination Port	State	Filter

4026531840	/bin/systemd	7295	7295	0	0x8938c945ec00	AF_INET	STREAM	TCP	192.168.0.2	59616	192.168.0.1	56999	ESTABLISHED	-
4026531840	/bin/systemd	7295	7295	3	0x8938c945f500	AF_INET	STREAM	TCP	0.0.0.0	1230	0.0.0.0	0	LISTEN	-
```

J'ai tenté de faire filtrage avec `grep` pour trouver le nom de domaine dans les différentes zones de mémoire utilisées par le processus /bin/systemd de pid 7295, mais je n'ai pas trouvé de piste exploitable pour trouver le nom de domaine associé à l'adresse IP.

Si le malware veut résoudre le nom domaine il doit surement faire une requete DNS, et pour trouver le nom de domaine associé à l'adresse IP, donc peut etre que l'information que nous cherchons n'est pas sur la forme de chaine de caractères, mais plutot sur la forme d'une requete DNS, et ce résultat ne peut etre stocké que dynamiquement dans le stack du processus, et c'est pour cela que je n'ai pas trouvé de piste exploitable pour trouver le nom de domaine associé à l'adresse IP dans les différentes zones de mémoire utilisées par le processus /bin/systemd de pid 7295.

Nous allons tenter d'écrire un script python pour analyser le stack du processus /bin/systemd de pid 7295 pour trouver des informations sur les requetes DNS effectuées par le processus, et c'est à partir de ces informations que nous allons pouvoir trouver le nom de domaine associé à l'adresse IP.

```python
from pathlib import Path

dump_file = "pid.7295.vma.0x7ffc628b9000-0x7ffc628da000.dmp"

data = Path(dump_file).read_bytes()

dns_entete = b"\x23\x15\x23\x15\x85\x80\x00\x01\x00\x01\x00\x00\x00\x00"


position = data.find(dns_entete)

print("Position du header DNS :", position)


if position >= 0:
    
    debut = max(0, position - 64)
    fin = min(len(data), position+100)

    extrait = data[debut:fin]
    print("\nContexte ASCII autour du paquet DNS :\n")

 
    lisible = ""
    for byte in extrait:
        if 32 <= byte <= 126:
            lisible += chr(byte)
        else:
            lisible += "."

    print(lisible)

else:
    print("Signature DNS non trouvée dans ce dump")
```

Resultat de l'execution du script :
```
onuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ python3 dns_scrypt1.py 
Position du header DNS : 118686

Contexte ASCII autour du paquet DNS :

..........._.b............._.b....._.b.....L@.............'.....#.#............helanhaiya.999apk.top................................................................
```

Conclusion : Le nom de domaine associé à l'adresse IP est helanhaiya.999apk.top.
