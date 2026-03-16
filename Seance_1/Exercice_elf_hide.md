Exercice elf_hide

Mot de passe à trouve : Is4ac a$im0vB

Méthode de résolution :

## Etape 1 : Analyse de l'entete du fichier

grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.

```bash
readelf -h elf_hide
``` 
resultat

```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse$ readelf -h elf_hide
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Intel 80386
  Version:                           0x1
  Entry point address:               0x12a4
  Start of program headers:          52 (bytes into file)
  Start of section headers:          12860 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         11
  Size of section headers:           40 (bytes)
  Number of section headers:         16
  Section header string table index: 15
```
Ce qui nous intéresse dnas l'entete:

- la classe `ELF32` qui nous indique que le fichier est un exécutable 32 bits
- le type `DYN` qui nous indique que le fichier est un exécutable position
- l'addresse d'entrée `0x12a4` qui nous indique ou commence le code exécutable

## Etape 2: Voir les sections du fichier ELF
grace à l'outils `readelf` avec l'option `-S` permet d'afficher ces informations.

```bash
readelf -S elf_hide
```
resultat:
```
[ 6] .text             PROGBITS        00001000 001000 0002b7 00  AX  0   0 16
```
Le flag `X` nous indique que cette section contient du code exécutable, et l'addresse `0x1000` nous indique ou commence ce code, qui correspond à l'adresse d'entrée de l'exécutable.



## Etape 3: Affichage du code assembleur de cette section
grace à l'outils `objdump` avec l'option `-d` permet d'afficher le code assembleur de cette section.
```bash
objdump -d elf_hide .text
```
En affichant le code assembleur pour trouver la fonction main à partir du point d'entrée du code exécutable, cela serait difficile  à code du format ELF32, car le code est embibé de notation `<main.c>`, en terme de l'isibilité du code assembleur, et aussi le code est optimisé, ce qui rend la lecture du code assembleur plus difficile.

Grace à l'outil `strip` qui permet de supprimer les symboles de debug du fichier ELF, et ainsi rendre le code assembleur plus lisible.

```bash
strip  elf_hide
```
Maintenant on peut afficher le code assembleur de cette section, et trouver la fonction main à partir du point d'entrée du code exécutable.

## Étape 3.2: Trouver le main et la chaine de caractères recherchée

### Etape 3.2.1: Analyse du point d'entrée du code exécutable

Après avoir aleger le code assembleur de la section .text, on peut trouver le point d'entrée du code exécutable à l'adresse `0x12a4`, et trouver la fonction main à l'adresse `0x1150`.

```bash
    12a4:	59                   	pop    %ecx
    12a5:	89 e3                	mov    %esp,%ebx
    12a7:	53                   	push   %ebx
    12a8:	51                   	push   %ecx
    12a9:	e8 a2 fe ff ff       	call   0x1150
    12ae:	89 c3                	mov    %eax,%ebx
    12b0:	b8 01 00 00 00       	mov    $0x1,%eax
    12b5:	cd 80                	int    $0x80
```

On ne peut pas affirmer que c'est la fonction main, mais on peut trouver la fonction qui compare les deux chaines de caractères, et trouver la chaine de caractères recherchée.

## Deplacement vers l'adresse `0x1150` pour trouver la comparaison des deux chaines de caractères


```bash
    1150:	55                   	push   %ebp
    1151:	89 e5                	mov    %esp,%ebp
    1153:	53                   	push   %ebx
    1154:	83 ec 24             	sub    $0x24,%esp
    1157:	e8 00 00 00 00       	call   0x115c
    115c:	58                   	pop    %eax
    115d:	81 c0 a4 2e 00 00    	add    $0x2ea4,%eax
    1163:	89 45 e0             	mov    %eax,-0x20(%ebp)
    1166:	8b 4d 0c             	mov    0xc(%ebp),%ecx
    1169:	8b 4d 08             	mov    0x8(%ebp),%ecx
    116c:	c7 45 f8 00 00 00 00 	movl   $0x0,-0x8(%ebp)
    1173:	8b 88 00 e0 ff ff    	mov    -0x2000(%eax),%ecx
    1179:	89 4d eb             	mov    %ecx,-0x15(%ebp)
    117c:	8b 88 04 e0 ff ff    	mov    -0x1ffc(%eax),%ecx
    1182:	89 4d ef             	mov    %ecx,-0x11(%ebp)
    1185:	8b 88 08 e0 ff ff    	mov    -0x1ff8(%eax),%ecx
    118b:	89 4d f3             	mov    %ecx,-0xd(%ebp)
    118e:	8a 80 0c e0 ff ff    	mov    -0x1ff4(%eax),%al
    1194:	88 45 f7             	mov    %al,-0x9(%ebp)
...
```
Ici on peut voir affectation sur le stack à l'adresse `-0xoffset(%ebp)` .
Ici on ajoute des données sur le stack de manière desordonnée, Vu que le stack va vers le bas , l'adresse qui nous intéresse `-0x15(%ebp)` est la plus grande adresse, et les autres sont des adresses plus petites, ce qui nous indique que les données sont ajoutées de manière desordonnée sur le stack.

Maintenant cherchons la source de ces données, et trouver la chaine de caractères recherchée.

Ces données sont ajoutées à partir de l'addresse relative `-0x2000(%eax)`, `-0x1ffc(%eax)`, `-0x1ff8(%eax)`, `-0x1ff4(%eax)`.

### Valeur de l'addresse pointée par eax

Si on regarde ici
```
  1157:	e8 00 00 00 00       	call   0x115c
    115c:	58                   	pop    %eax
    115d:	81 c0 a4 2e 00 00    	add    $0x2ea4,%eax
```

On peut voir call `0x115c` va directement après le call, le call pousse cette adreese sur le stack avec `push 0x115c`.
 Après le pop va  ajouté `0x115c` à `eax`,  et après l'addition de `0x2ea4` à `eax`, on va trouver l'addresse relative qui pointe vers les données que nous cherchons.

Donc `eax` = `0x2ea4` + `0x115c` = `0x4000`

Vu que `eax` pointe vers l'addresse `0x4000`, et que les données sont ajoutées à partir de l'addresse relative `-0x2000(%eax)`, `-0x1ffc(%eax)`, `-0x1ff8(%eax)`, `-0x1ff4(%eax)`, on peut trouver l'addresse absolue de ces données.

#### Etape 3.2.6: Afficher la valeur de l'addresse relative

```bash
0x2000:     0x0b    0x31    0x76    0x23
0x2004:     0x21    0x62    0x23    0x66
0x2008:     0x2b    0x2f    0x72    0x34
0x200c:     0x00    0x4e    0x65    0x65
```

#### Etape 3.2.7: Trouver la chaine de caractères recherchée

Comparaison que donne toujours le meme resultat, Obfuscation>

```bash
 11c1:	8b 5d e0             	mov    -0x20(%ebp),%ebx
    11c4:	8b 45 0c             	mov    0xc(%ebp),%eax
    11c7:	8b 48 04             	mov    0x4(%eax),%ecx
    11ca:	8d 83 30 e0 ff ff    	lea    -0x1fd0(%ebx),%eax
    11d0:	89 0c 24             	mov    %ecx,(%esp)
    11d3:	89 44 24 04          	mov    %eax,0x4(%esp)
    11d7:	e8 64 fe ff ff       	call   0x1040
    11dc:	83 f8 00             	cmp    $0x0,%eax
    11df:	0f 85 20 00 00 00    	jne    0x1205
```

Si on calcule 0x4000-0x1fd0 = 0x2030, on trouve l'addresse de la chaine de caractères attendue qui un piege, pour nous dire `Nice try!` parce que c'est un piege tendu pour nous faire croire que la chaine de caractères recherchée est `Isaac Asimov`.

```
 2030 49736161 63204173 696d6f76 004e6963  Isaac Asimov.Nic
 2040 65207472 79210a00 476f6f64 20706173  e try!..Good pas
```


### Allons à l'adresse `0x1205` pour trouver la comparaison des deux chaines de caractères


```bash
1205:	c7 45 e4 00 00 00 00 	movl   $0x0,-0x1c(%ebp)
    120c:	8b 45 0c             	mov    0xc(%ebp),%eax
    120f:	8b 40 04             	mov    0x4(%eax),%eax
    1212:	8b 4d e4             	mov    -0x1c(%ebp),%ecx
    1215:	0f be 04 08          	movsbl (%eax,%ecx,1),%eax
    1219:	83 f8 00             	cmp    $0x0,%eax
    121c:	0f 84 31 00 00 00    	je     0x1253
    1222:	8b 45 0c             	mov    0xc(%ebp),%eax
    1225:	8b 40 04             	mov    0x4(%eax),%eax
    1228:	8b 4d e4             	mov    -0x1c(%ebp),%ecx
    122b:	0f be 04 08          	movsbl (%eax,%ecx,1),%eax
    122f:	83 f0 42             	xor    $0x42,%eax
    1232:	25 ff 00 00 00       	and    $0xff,%eax
    1237:	88 c2                	mov    %al,%dl
    1239:	8b 45 0c             	mov    0xc(%ebp),%eax
    123c:	8b 40 04             	mov    0x4(%eax),%eax
    123f:	8b 4d e4             	mov    -0x1c(%ebp),%ecx
    1242:	88 14 08             	mov    %dl,(%eax,%ecx,1)
    1245:	8b 45 e4             	mov    -0x1c(%ebp),%eax
    1248:	83 c0 01             	add    $0x1,%eax
    124b:	89 45 e4             	mov    %eax,-0x1c(%ebp)
    124e:	e9 b9 ff ff ff       	jmp    0x120c
```
Ici on peut voir une iteration sur la chaine passée en argument, et une comparaison de chaque octet de cette chaine avec le caractère `\0`, si la comparaison est égale à 0, cela signifie que la fin de la chaine a été atteinte, sinon on applique un `xor 42` sur chaque octet de cette chaine, et on stocke le résultat dans la même adresse de la chaine passée en argument.


Ici la chaine passée en argument a été irité , en aapliquant un `xor 42`, sur chaque octet de cette chaine.

```
argv[i] ^= 0x42
```

En appliquant l'égalite A `xor` B = C <=>  A = B `xor` C

argi[i] = 0x42 `xor` secret[i]

d'ou la chaine de caractères recherchée est:

```
0x0b `xor` 0x42 = 0x49 = 'I' 
0x31 `xor` 0x42 = 0x73 = 's'
0x76 `xor` 0x42 = 0x34 = '4'
0x23 `xor` 0x42 = 0x61 = 'a'
0x21 `xor` 0x42 = 0x63 = 'c'


0x62 `xor` 0x42 = 0x20 = ' '
0x23 `xor` 0x42 = 0x61 = 'a'
0x66 `xor` 0x42 = 0x24 = '\$'
0x2b `xor` 0x42 = 0x69 = 'i'
0x2f `xor` 0x42 = 0x6d = 'm'
0x72 `xor` 0x42 = 0x30 = '0'
0x34 `xor` 0x42 = 0x76 = 'v'
0x00 `xor` 0x42 = 0x42 = 'B'
0x4e `xor` 0x42 = 0x0c = '\0'

Is4ac a\$im0vB
```





