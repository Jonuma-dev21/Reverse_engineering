Exercice got_pois

Mot de passe à trouve : BurYrBDay R4

Méthode de résolution :

## Etape 1 : Analyse de l'entete du fichier

grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.

```bash
readelf -h got_pois
``` 
resultat

```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse$ readelf -h got_pois 
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1060
  Start of program headers:          64 (bytes into file)
  Start of section headers:          11688 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         12
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```


Ce qui nous intéresse dnas l'entete:

- la classe `ELF64` qui nous indique que le fichier est un exécutable 64 bits
- le type `DYN` qui nous indique que le fichier est un exécutable position
- l'addresse d'entrée `0x1060` qui nous indique ou commence le code exécutable

## Etape 2: Voir les sections du fichier ELF

grace à l'outils `readelf` avec l'option `-S` permet d'afficher ces informations.

```bash
readelf -S got_pois
```

resultat

```
[15] .text             PROGBITS         0000000000001080  00001080
       0000000000000264  0000000000000000  AX       0     0     16
```
Le flag `X` nous indique que cette section contient du code exécutable, et l'addresse `0x1080` nous indique ou commence ce code, qui correspond à l'adresse d'entrée de l'exécutable.

## Etape 3: Analyse de l'entrée du code exécutable

grace à l'outils `objdump` avec l'option `-d` permet d'afficher le code assembleur de cette section.

```bash
objdump -d got_pois .text
``` 
dans ce code aasembleur, ce qui nous intéresse c'est le point d'entrée du code exécutable, qui correspond à l'addresse `0x1060` et qui nous permettra de trouver la partie du code qui compare les deux chaines de caractères.

ce point d'entrée correspond à l'étiquette `_start` dans le code assembleur.

### Etape 3.1: Analyse du code assembleur de la fonction `_start`
dans cette partie du code, on peut voir l'addresse de la l'étiquette de la fonction main `0x1c10` qui est passée en argument pour la fonction lib_c_start_main, qui est la fonction qui lance l'exécution du code de la fonction main.


```assembly
  1078:	48 8d 3d 91 0b 00 00 	lea    0xb91(%rip),%rdi        # 1c10 <main>
    107f:	ff 15 33 24 00 00    	call   *0x2433(%rip)        # 34b8 <__libc_start_main@GLIBC_2.34>
```

### Etape 3.2: Analyse du code assembleur de la fonction main

En analysant le code du main de fichier `got_pois`, Je remarque une fonction `<fixgot>` qui a été sans condition, et qui est appelée avant la comparaison des deux chaines de caractères.

#### 3.2.1 Analyse de la fonction `<fixgot>`

Tout au début on peut remarquer que le registre `rax` a été affecté à une addresse mémoire dont la valeur est la somme `0x1cf5 + 0x17c3` qui donne l'addresse `0x34b8` qui correspond à l'adresse de la fonction `__libc_start_main@GLIBC_2.34` dans la table des symboles dynamiques, et qui est la fonction qui lance l'exécution du code de la fonction main.
```
bb4:	48 8d 05 3a 01 00 00 	lea    0x13a(%rip),%rax        # 1cf5 <_etext>
    1bbb:	48 05 c3 17 00 00    	add    $0x17c3,%rax
```
Hors l'addresse des fonctions sont stockées dans la table des symboles dynamiques.

Voyons ou l'addresse de cettte table GOT avec cette commande:

```bash
objdump -s -j .got got_pois
```
Resultat

```
onuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse$ objdump -s -j .got got_pois

got_pois:     file format elf64-x86-64

Contents of section .got:
 34b8 00000000 00000000 00000000 00000000  ................
 34c8 00000000 00000000 00000000 00000000  ................
 34d8 00000000 00000000 00000000 00000000  ................
```

C'est bien l'addresse de la table GOT.

en analysant le reste du code j'ai pu remarquer qui a une itération sur cette table dans le but de trouver l'addresse de la fonction `strcmp` qui est la fonction qui compare les deux chaines de caractères. 
Chargée ici:
```
1bdd:	48 8b 15 e4 18 00 00 	mov    0x18e4(%rip),%rdx        # 34c8 <strcmp@GLIBC_2.2.5>
```

Quand cette fonction est toruvée, elle est remplacée par l'addresse d'une autre foncction, la fonction `<mymy_strcmp_my>` à l'addrese `0x14c0`.

#### 3.2.2 Analyse de la fonction `<mymy_strcmp_my>`

En analysant le code, j'ai pu remarquer qu'avant de faire la comparaison, il vérifie si les deux chaines font 12 caractères, et si c'est le cas, il fait la comparaison, sinon il retourne une valeur différente de 1 pour indiquer que les deux chaines sont différentes.
Extrait du code:

```
    198a:	48 83 7d e0 0c       	cmpq   $0xc,-0x20(%rbp)
    198f:	0f 85 0b 00 00 00    	jne    19a0 <mymy_strcmp_mymy+0x40>
    1995:	48 83 7d d8 0c       	cmpq   $0xc,-0x28(%rbp)
    199a:	0f 84 0c 00 00 00    	je     19ac <mymy_strcmp_mymy+0x4c>
    19a0:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
```

Ensuite, il utlise cet algo pour comparer les deux chaines de caractères:

```
 s1[0] vs s2[8]
s1[1] vs s2[9]
s1[2] vs s2[10]
s1[3] vs s2[11]
s1[4] vs s2[5]
s1[5] vs s2[4]
s1[6] vs s2[7]
s1[7] vs s2[6]
s1[8] vs s2[2]
s1[9] vs s2[3]
s1[10] vs s2[0]
s1[11] vs s2[1]
``` 

La chaine stockée en mémoire:

"R4y BraDBurY"

en appliquant l'algo de comparaison, on trouve que la chaine comparée est:

"BurYrBDay R4"


