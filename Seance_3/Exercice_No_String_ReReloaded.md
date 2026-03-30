Exercice : No String ReReloaded

Mot de passe à trouve : "DOUG145 4d4m5"

Méthode de résolution : 

Dans cet exercice nous allons devoir faire une étude de la chaine de manière dynamique pour le password.

### Étape 1 : Exécution du programme

En €xecutant le programme on peut voir que celui ci attend une et une seule entrée de la part de l'utilisateur, et après l'entrée du mot de passe le programme affiche un message d'erreur "Try again" et se termine.

### Étape 2 : Analyse du code avec ltrace

ltrace est un outil de traçage de bibliothèques partagées qui permet de suivre les appels de fonctions et les interactions avec les bibliothèques partagées utilisées par un programme.

Avec ltrace on n'arrive pas trouver la fonction de comparaison des chaines de caracteres.

### Étape 3 : Analyse du code avec strace
strace est un outil de traçage de système qui permet de suivre les appels système effectués par un programme.

Avec strace je n'ai pas trouvé de piste exploitable pour trouver la chaine de caractères du mot de passe.

### Étape 4 : Analyse du code avec gdb
gdb est un débogueur qui permet d'analyser le code binaire d'un programme en temps réel, et de suivre l'exécution du programme pas à pas.

Mais pour pouvoir mieux analyser le code avec gdb, j'ai décidé de faire un reverse du binaire avec ghidra pour trouver la fonction de comparaison des chaines de caractères, grace a son adresse, et c'est à partir de cette fonction que je vais pouvoir trouver la chaine de caractères du mot de passe.

#### Étape 4.1 : Reverse du binaire avec ghidra

En analysant le code avec ghidra j'ai trouvé l'adresse de l'appel à la fonction de comparaison des chaines de caractères, et c'est à partir de cette adresse que je vais pouvoir peut etre trouver la chaine de caractères du mot de passe si elle est en claire.

mais étant donné que gdb et ghidra n'utlise pas la même base d'adresse pour le binaire, je vais modifer la base d'adresse dans ghidra pour qu'elle soit la même que celle de gdb, et c'est à partir de cette base d'adresse que je vais pouvoir trouver la chaine de caractères du mot de passe.

#### Étape 4.1.1 trouver la base d'adresse de gdb

```bash
(gdb) starti
Starting program: /home/jonuma/Documents/Master1/S2/Cours/SDS/reverse_dynamic/nsrr mman

Program stopped.
0x00007ffff7fe4540 in _start () from /lib64/ld-linux-x86-64.so.2
(gdb) info proc mappings 
process 5569
Mapped address spaces:

          Start Addr           End Addr       Size     Offset  Perms  objfile
      0x555555554000     0x5555555b7000    0x63000        0x0  r--p   /home/jonuma/Documents/Master1/S2/Cours/SDS/reverse_dynamic/nsrr
```

#### Étape 4.1.2 Modifier la base d'adresse dans ghidra
la base d'adresse de gdb est 0x555555554000, et la base d'adresse de ghidra est 0x100000, donc je vais modifier la base d'adresse de ghidra pour qu'elle soit la même que celle de gdb, et c'est à partir de cette base d'adresse que je vais pouvoir trouver la chaine de caractères du mot de passe.

#### Étape 4.2 : Trouver l'appel à la fonction de comparaison des chaines de caractères
```bash
    5555555553c4 e8 37 fe        CALL       FUN_555555555200                                 undefined FUN_555555555200()
                 ff ff
    5555555553c9 83 f8 00        CMP        EAX,0x0
    5555555553cc 0f 85 13        JNZ        LAB_5555555553e5
                 00 00 00
    5555555553d2 48 8d 3d        LEA        RDI,[s_Well_done_55555555604c]                   = "Well done\n"
                 73 0c 00 00
```
on va mettre un breakpoint, a l'adresse 0x5555555553c4, sur l'appel à la fonction de comparaison des chaines de caractères, et c'est à partir de ce breakpoint que je vais pouvoir trouver la chaine de caractères du mot de passe.


#### Étape 4.3 : Trouver la chaine de caractères du mot de passe
```bash
(gdb) b *0x5555555553c4
```

Apres avoir mis le breakpoint on fait un run du programme, et on arrive au breakpoint, et c'est à partir de ce breakpoint que je vais pouvoir trouver la chaine de caractères du mot de passe.

```bash
(gdb) run 
```

Apres avoir fait un run du programme, on arrive au breakpoint, et c'est à partir de ce breakpoint que je vais pouvoir trouver la chaine de caractères du mot de passe.

mais etant donné que la chaine est passé en argument(2eme argument) via le registre `RSI`, je vais afficher le contenu de ce registre pour trouver la chaine de caractères du mot de passe.

```bash
Breakpoint 1, 0x00005555555553c4 in ?? ()
(gdb) x/s $rsi
0x7fffffffd880: "DOUG145 4d4m5"
```

Ainsi le mot de passe est "DOUG145 4d4m5", et en l'entrant dans le programme on peut voir que celui ci affiche un message de succès "Well done" et se termine.