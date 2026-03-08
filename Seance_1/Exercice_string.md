Exercice string

Mot de passe à trouve : Ors0n Sco77 C4rd

Méthode de résolution :

## Etape 1 : Analyse de l'entete du fichier

grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.

```bash
readelf -h string
``` 
resultat

```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse$ readelf -h string 
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
  Start of section headers:          14096 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30

```
Ce qui nous intéresse dnas l'entete:

- la classe `ELF64` qui nous indique que le fichier est un exécutable 64 bits
- le type `DYN` qui nous indique que le fichier est un exécutable position
- l'addresse d'entrée `0x1080` qui nous indique ou commence le code exécutable

## Etape 2 : informations sur la section éxécutable
Avant de commencer, il faut que j'aie un minimum d'informations sur le fichier pour savoir dans quelle section se trouve son code source.

avec l'outils `readelf` avec l'option `-S` permet d'afficher ces informations.

```bash
readelf -S string
```

resultat
```
 .text             PROGBITS         0000000000001060  00001060
       0000000000000178  0000000000000000  AX       0     0     16
```

Ou on peut voir le flag `X` qui confirme la présence de codes sources éxécutables dans cette section.

## Etape 3: Analyse du code source de la section éxécutable
grace à l'outils `objdump` avec l'option `-d` permet d'afficher le code assembleur de cette section.

commande:
```bash
objdump -d string .text
```
Chechons le point d'entrée du code éxécutable, qui est à l'adresse `0x1060`, et analysons le code à partir de cette adresse.

Le point d'entrée du code éxécutable est à l'adresse `0x1060`, et en analysant le code à partir de cette adresse, on peut voir que c'est la fonction `_start` qui est appelée.

### Etape 3.1: Analyse du code assembleur de la fonction `_start`
dans cette partie du code, on peut voir l'addresse de la l'étiquette de la fonction main `1150` qui est passée en argument pour la fonction lib_c_start_main, qui est la fonction qui lance l'exécution du code de la fonction main.


```assembly
  1078:	48 8d 3d d1 00 00 00 	lea    0xd1(%rip),%rdi        # 1150 <main>
    107f:	ff 15 53 2f 00 00    	call   *0x2f53(%rip)        # 3fd8 <__libc_start_main@GLIBC_2.34>

```

### Etape 3.2: Analyse du code assembleur de la fonction main

Analysons le code de la fonction main, et cherchons la partie du code qui compare les deux chaines de caractères. En regardant ici, on peut voir la comparaison des deux chaines.

```

 1192:       48 8d 35 81 0e 00 00    lea    0xe81(%rip),%rsi      #       201a <_IO_stdin_used+0x1a>
 1199:       e8 a2 fe ff ff          call   1040 <strcmp@plt>
```

`0x201a` représente l'addresse de la chaines comparée.

Etape 3: trouvée la chaine à l'adresse `0x201a`.

avec la commande 
```
objdump -s string
```

on peut voir toute les sections avec leurs adresses memoire et nous, on cherche la chiane commencant à l'addresse `0x201a`

```

 2010 61737377 6f72643e 0a004f72 73306e20  assword>..Ors0n 
 2020 53636f37 37204334 72640047 6f6f6420  Sco77 C4rd.Good 
 2030 70617373 776f7264 0a005772 6f6e6720  password..Wrong 
```

notre chaine commence c'est tout cela en hexa

```
4f72 73306e20 2020 53636f37 37204334 726400
```

Qui correspond au final à la chaine `Ors0n Sco77 C4rd`