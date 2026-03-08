Exercice stat_link

Mot de passe à trouve : 1984

Méthode de résolution :

## Etape 1 : Analyse de l'entete du fichier

grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.

```bash
readelf -h stat_link
``` 
resultat

```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse$ readelf -h stat_link 
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 03 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - GNU
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x4018c0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          951280 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         10
  Size of section headers:           64 (bytes)
  Number of section headers:         27
  Section header string table index: 26

```
Ce qui nous intéresse dnas l'entete:

- la classe `ELF64` qui nous indique que le fichier est un exécutable 64 bits
- le type `EXEC` qui nous indique que le fichier est un exécutable
- l'addresse d'entrée `0x4018c0` qui nous indique ou commence le code exécutable

## Etape 2: Voir les sections du fichier ELF

grace à l'outils `readelf` avec l'option `-S` permet d'afficher ces informations.

```bash
readelf -S stat_link
```

resultat
```
[15] .text             PROGBITS         0000000000001060  00001060
       0000000000000c85  0000000000000000  AX       0     0     16
```

Le flag `X` nous indique que cette section contient du code exécutable, et l'addresse `0x1060` nous indique ou commence ce code, qui correspond à l'adresse d'entrée de l'exécutable.

Suite à cette commande, on peut remqarquer l'absence de symboles pour trouver les fonctions utilisées dans le code, grace à l'outils `readelf` avec l'option `-s` permet d'afficher ces informations.

```bash
readelf -s stat_link
```
resultat vide, les symboles sont supprimés, ce qui rend l'analyse du code plus difficile.

## Etape 3: Affichage du code assembleur de cette section
grace à l'outils `objdump` avec l'option `-d` permet d'afficher le code assembleur de cette section.

```bash
objdump -d stat_link .text > stat_link.asm
``` 
dans ce code aasembleur, ce qui nous intéresse c'est le point d'entrée du code exécutable, qui correspond à l'addresse `0x4018c0` et qui nous permettra de trouver la partie du code qui compare les deux chaines de caractères.


### Etape 3.1: Trouver la partie du code qui lance l'exécution de la fonction main

Maintenant, il faut trouver la partie qui sera surement dans la fonction main, qui est souvent passer en argument pour la fonction `__libc_start_main` qui est la fonction qui lance l'exécution du code de la fonction main. 
Mais qu'on doit trouver sans l'étiquette de la fonction main, car les symboles sont supprimés.
qu'on peut voir ici:

```
4018d8:	48 c7 c7 f0 19 40 00 	mov    $0x4019f0,%rdi
4018df:	67 e8 1b 14 00 00    	addr32 call 0x402d00
```

notre adreese recherché est `0x4019f0` qui correspond à l'adresse de la fonction main, et qui est passé en argument pour la fonction `__libc_start_main` qui est appelé à l'adresse `0x402d00`.

### Etape 3.2: Analyse du lancement de la fonction main
quand on lance le programme la fonction `printf` est appelée pour demander d'entrée le nombre secret, donc la fonction scanf a été appellée pour lire l'entrée et la fonction `printf` est appelée deux fois pour afficher le message de succès ou d'échec, et la fonction de comparaison du nombre secret est appelée pour comparer l'entrée avec la valeur attendue.


### Etape 3.3: Analyse du code assembleur de la fonction main pour trouver la chaine de caractères recherchée

Après avoir trouvé le point d'entrée de la fonction main `0x4019f0`, il faut trouver la partie du code qui compare les deux nombres, et

donc on doit trouvé l'entrée de la fonction scanf qui lit l'entrée de l'utilisateur, et la fonction de comparaison qui compare cette entrée avec la valeur attendue, et trouver.

Logiquemt cette fonction doit etre appellée au millieu de plusieurs appels à la fonction `printf` qui affiche les messages de succès ou d'échec, et qui affiche aussi le message demandant d'entrée le nombre secret.

ce qui correspond à cette fonction:

```
lea    0xb4601(%rip),%rdi        # 0x4b6015
  401a14:	48 8d 75 f8          	lea    -0x8(%rbp),%rsi
  401a18:	b0 00                	mov    $0x0,%al
  401a1a:	e8 f1 9f 00 00       	call   0x40ba10
  401a1f:	8b 45 f8             	mov    -0x8(%rbp),%eax
```

la variable locale à l'adresse `-0x8(%rbp)` est utilisée pour stocker l'entrée de l'utilisateur.

suite du code

```
 401a22:	35 23 23 00 00       	xor    $0x2323,%eax
 401a27:	3d e3 24 00 00       	cmp    $0x24e3,%eax
```

On va utliser cette proprieté de l'operateur `xor` pour trouver la valeur attendue.

car, A `xor` B = C <=>  A = B `xor` C

valeur = 0x2323 `xor` 0x24e3 = 0x7c0

La valeur attendue est `0x7c0` qui correspond à `1984` en décimal, et c'est le nombre secret à trouver.


