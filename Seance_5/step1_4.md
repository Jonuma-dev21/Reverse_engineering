Exercice : Step 1/4 Botnet

Mot de passe à trouve : "12661c9b284e6993e43a7dad7a987008da86f582bc6d4958e24ad39acef644dd"

Méthode de résolution : 

Contexte du piratage  :

Une machine virtuel a été compromise par un attaquant qui a réussi à s'introduire dans le système et a laissé un malware qui a permis à l'attaquant de prendre le contrôle de la machine. Nous avons réussi à récupérer une image de la mémoire de la machine compromise.

Objectif : `Locate and dump the malicious process. Give the sha256 of the dumped ELF.`


## Étape 1 : Analyse de l'image de la mémoire avec volatility3

Pour commencer nous allons tenter de trouver le processus malvaillant en analysant les processus étaint en cours d'exécution au moment de la capture de l'image de la mémoire, pour cela nous allons utiliser volatility3 qui est un framework d'analyse de la mémoire qui permet d'analyser les images de la mémoire et de trouver des informations sur les processus en cours d'exécution, les modules chargés, les connexions réseau,...

## Etape 1.2 : Liste des processus en cours d'exécution

```bash
python3 vol.py -f ./../memoire/ram.img linux.pslist.PsList
```
lien vers le resultat de la commande : [pslist.txt](./extrait_ps.txt)

Extrait de la liste des processus en cours d'exécution qui nous intéresse :
```
0x8938c68f1980	7295	7295	1322	/bin/systemd	0	0	0	0	2023-12-01 18:54:02.935400 UTC	Disabled
0x89386c493300	7296	7296	7295	/bin/systemd	0	0	0	0	2023-12-01 18:54:02.935675 UTC	Disabled
```

Dans cette affichage j'ai pu constaté deux informations un peu louches:

- 1 : dans les systèmes linux `/bin/systemd` est souvent un lien symbolique vers `/lib/systemd/systemd` qui est un processus de base du système d'exploitation, mais dans notre cas on voit que le processus 7295 et 7296 sont des processus qui ont pour chemin d'executable `/bin/systemd` ce qui est un peu suspect.

- 2 : les processus 7295 et 7296 ont été lancés à la même date et à la même heure, ce qui est aussi un peu suspect. Hors ces processus systèmes sont souvnet lancés au démarrage du système, et il est rare de les voir être lancés à la même date et à la même heure. Et avec des pid aussi élevé que 7295 et 7296, ce qui est aussi un peu suspect.

## Etape 1.3 : Analyse des processus suspects pour trouver le processus malvaillant

Nous allons commnencer par dumper les processus 7295 et 7296 pour voir s'ils contiennent des informations intéressantes, pour cela nous allons utiliser le plugin `linux.proc.Maps` qui permet de trouver les différentes zones de mémoire utilisées par un processus, et de les dumper pour les analyser par la suite.

### Dump du processus 7295

Commande utilisée pour dumper le processus 7295 :
```bash
python3 vol.py -f ./../memoire/ram.img linux.proc.Maps --pid 7295 --dump
```

Et on a eu le résultat suivant :
```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ python3 vol.py -f ./../memoire/ram.img linux.proc.Maps --pid 7295 --dump
Volatility 3 Framework 2.28.1
Progress:  100.00		Stacking attempts finished            
PID	Process	Start	End	Flags	PgOff	Major	Minor	Inode	File Path	File output

7295	/bin/systemd	0x400000	0x410000	r-x	0x0	8	3	263912	/home/host/Downloads/bin/systemd	pid.7295.vma.0x400000-0x410000.dmp
7295	/bin/systemd	0x50f000	0x511000	rw-	0xf000	8	3	263912	/home/host/Downloads/bin/systemd	pid.7295.vma.0x50f000-0x511000.dmp
7295	/bin/systemd	0x511000	0x513000	rw-	0x0	0	0	0	Anonymous Mapping	pid.7295.vma.0x511000-0x513000.dmp
7295	/bin/systemd	0x11df000	0x11e0000	rw-	0x0	0	0	0	[heap]	pid.7295.vma.0x11df000-0x11e0000.dmp
7295	/bin/systemd	0x7ffc628b9000	0x7ffc628da000	rw-	0x0	0	0	0	[stack]	pid.7295.vma.0x7ffc628b9000-0x7ffc628da000.dmp
7295	/bin/systemd	0x7ffc62934000	0x7ffc62938000	r--	0x0	0	0	0	Anonymous Mapping	pid.7295.vma.0x7ffc62934000-0x7ffc62938000.dmp
7295	/bin/systemd	0x7ffc62938000	0x7ffc6293a000	r-x	0x0	0	0	0	[vdso]	pid.7295.vma.0x7ffc62938000-0x7ffc6293a000.dmp
```

### Analyse des dumps du processus 7295
En regardant le resultat du dump du processus 7295, on peut voir que le processus 7295 a plusieurs zones de mémoire qui sont utilisées, mais la zone de mémoire qui nous intéresse le plus est la zone de mémoire qui commence à l'adresse 0x400000 et qui se termine à l'adresse 0x410000, cette zone de mémoire est une zone de mémoire exécutable (r-x) qui contient le code du processus, et c'est à partir de cette zone de mémoire que nous allons pouvoir trouver le code malvaillant du processus.

### type de fichier du dump de la zone de mémoire 0x400000-0x410000

```bash 
file pid.7295.vma.0x400000-0x410000.dmp
```
Résultat de la commande :
```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ file pid.7295.vma.0x400000-0x410000.dmp
pid.7295.vma.0x400000-0x410000.dmp: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, missing section headers at 66368
```

On peut voir que le dump de la zone de mémoire 0x400000-0x410000 est un fichier ELF 64-bit qui est un format de fichier exécutable utilisé par Linux.

### sha256 du dump de la zone de mémoire 0x400000-0x410000

```bash
sha256sum pid.7295.vma.0x400000-0x410000.dmp
```
Résultat de la commande :
```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ sha256sum pid.7295.vma.0x400000-0x410000.dmp
4ee71841b003b87f70942cf40c054cb4b9f22a4e37ce0c66a19137930d23682d  pid.7295.vma.0x400000-0x410000.dmp
```

D'après les instrcutions de l'exercice, si le hash commence par 4ee71841b003b87f709..., alors le fichier est incomplet, et il faut le reconstruire en concaténant les dumps des différentes zones de mémoire du processus 7295, pour cela nous allons concaténer les dumps des zones de mémoire 0x400000-0x410000, 0x50f000-0x511000.

```bash
cat pid.7295.vma.0x400000-0x410000.dmp pid.7295.vma.0x50f000-0x511000.dmp > final.elf
```

Puis on va calculer le sha256 du fichier final.elf pour voir si on a réussi à reconstruire le fichier correctement.

```bash
sha256sum final.elf
```

Résultat de la commande :

```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ sha256sum final.elf
12661c9b284e6993e43a7dad7a987008da86f582bc6d4958e24ad39acef644dd  final.elf
```

D'ou le hash recherché est : `12661c9b284e6993e43a7dad7a987008da86f582bc6d4958e24ad39acef644dd`
