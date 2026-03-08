Exercice nostring

Mot de passe à trouve : Ar7hUr c. Cl4rk3

Méthode de résolution :

## Etape 1 : Analyse de l'entete du fichier

grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.

```bash
readelf -h nostring
``` 
resultat

```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse$ readelf -h nostring 
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
  Entry point address:               0x1080
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14232 (bytes into file)
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

## Etape 2: Voir les sections du fichier ELF

grace à l'outils `readelf` avec l'option `-S` permet d'afficher ces informations.

```bash
readelf -S nostring
```

resultat

```
[15] .text             PROGBITS         0000000000001080  00001080
       0000000000000264  0000000000000000  AX       0     0     16
```
Le flag `X` nous indique que cette section contient du code exécutable, et l'addresse `0x1080` nous indique ou commence ce code, qui correspond à l'adresse d'entrée de l'exécutable.

## Etape 3: Affichage du code assembleur de cette section
grace à l'outils `objdump` avec l'option `-d` permet d'afficher le code assembleur de cette section.

```bash
objdump -d nostring .text
``` 
dans cereadelf -h elf_hide code aasembleur, ce qui nous intéresse c'est le point d'entrée du code exécutable, qui correspond à l'addresse `0x1080` et qui nous permettra de trouver la partie du code qui compare les deux chaines de caractères.

ce point d'entrée correspond à l'étiquette `_start` dans le code assembleur.

### Etape 3.1: Analyse du code assembleur de la fonction `_start`
dans cette partie du code, on peut voir l'addresse de la l'étiquette de la fonction main `1200` qui est passée en argument pour la fonction lib_c_start_main, qui est la fonction qui lance l'exécution du code de la fonction main.


```assembly
1098:	48 8d 3d 61 01 00 00 	lea    0x161(%rip),%rdi        # 1200 <main>
    109f:	ff 15 33 2f 00 00    	call   *0x2f33(%rip)        # 3fd8 <__libc_start_main@GLIBC_2.34>
```

### Etape 3.2: Analyse du code assembleur de la fonction main

En analysant le code du main de fichier `nostring`, il est presque identique à son homologue `string`, mais la différence réside sur le fait que l'argument passé la fonction de comparaison de chaines est le pointeur d'une variable locale à main.

qu'on peut voir dans cette extrait:

```
  12a1:	48 8d 75 d0          	lea    -0x30(%rbp),%rsi
```

chaine qui a été initialisé ici

```
    1259:	c6 45 d0 41          	movb   $0x41,-0x30(%rbp)
    125d:	c6 45 d1 72          	movb   $0x72,-0x2f(%rbp)
    1261:	c6 45 d2 37          	movb   $0x37,-0x2e(%rbp)
    1265:	c6 45 d3 68          	movb   $0x68,-0x2d(%rbp)
    1269:	c6 45 d4 55          	movb   $0x55,-0x2c(%rbp)
    126d:	c6 45 d5 72          	movb   $0x72,-0x2b(%rbp)
    1271:	c6 45 d6 20          	movb   $0x20,-0x2a(%rbp)
    1275:	c6 45 d7 63          	movb   $0x63,-0x29(%rbp)
    1279:	c6 45 d8 2e          	movb   $0x2e,-0x28(%rbp)
    127d:	c6 45 d9 20          	movb   $0x20,-0x27(%rbp)
    1281:	c6 45 da 43          	movb   $0x43,-0x26(%rbp)
    1285:	c6 45 db 6c          	movb   $0x6c,-0x25(%rbp)
    1289:	c6 45 dc 34          	movb   $0x34,-0x24(%rbp)
    128d:	c6 45 dd 72          	movb   $0x72,-0x23(%rbp)
    1291:	c6 45 de 6b          	movb   $0x6b,-0x22(%rbp)
    1295:	c6 45 df 33          	movb   $0x33,-0x21(%rbp)
```

qui correspond à la chaine `Ar7hUr c. Cl4rk3` qui est comparé à la chaine `Ors0n Sco77 C4rd` dans le fichier `string`, et qui est différent de celle ci, ce qui explique pourquoi le mot de passe n'est pas trouvé dans ce fichier.




