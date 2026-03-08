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
En affichant le code assembleur ou trouver la fonction main à partir du point serais difficile  à code du format ELF32.

L'idée m'est venue d'utliser l'outil `gdb` pour faire du reverse engineering dynamique, et ainsi trouver la partie du code qui compare les deux chaines de caractères.

### Etape 3.2.1: Lancement de gdb
```bash
gdb ./elf_hide
```

### Etape 3.2.2: lance la première instruction du code exécutable
```bash
starti
```
### Etape 3.2.3:  Trouver l'addresse de base du programme
```bash
info proc mappings
```
resultat:
```
Mapped address spaces:

        Start Addr   End Addr       Size     Offset  Perms   objfile
        0x56555000 0x56556000     0x1000        0x0  r--p   /home/jonuma/Documents/Master
```
Grace à cette commande, on peut trouver l'addresse de base du programme qui est `0x56555000`, et ainsi trouver l'addresse de la fonction main qui est `0x56555000 + 0x12a4 = 0x565562a4`.

### Etape 3.2.4: Afficher le code assembleur de la fonction main à du point d'entrée du code exécutable

déplacons le pointeur d'exécution à l'addresse de la fonction main, et affichons le code assembleur à partir de cette addresse.
```bash
break *0x565562a4
continue
x/20i rip
```

resultat:
```
 0x565562a7 <main.c>: push   %ebx
   0x565562a8 <main.c>: push   %ecx
   0x565562a9 <main.c>: call   0x56556150 <main.c>
```
maintenant on va sauter l'appel de la fonction main, et afficher le code assembleur à partir de l'addresse de la fonction main.
```bash
break *0x56556150
continue
x/20i rip
```
resultat:

```
(gdb) x/80i $eip
=> 0x56556150 <main.c>: push   %ebp
   0x56556151 <main.c>: mov    %esp,%ebp
   0x56556153 <main.c>: push   %ebx
   0x56556154 <main.c>: sub    $0x24,%esp
   0x56556157 <main.c>: call   0x5655615c <main.c>
   0x5655615c <main.c>: pop    %eax
   0x5655615d <main.c>: add    $0x2ea4,%eax
   0x56556163 <main.c>: mov    %eax,-0x20(%ebp)
   0x56556166 <main.c>: mov    0xc(%ebp),%ecx
   0x56556169 <main.c>: mov    0x8(%ebp),%ecx
   0x5655616c <main.c>: movl   $0x0,-0x8(%ebp)
   0x56556173 <main.c>: mov    -0x2000(%eax),%ecx
   0x56556179 <main.c>: mov    %ecx,-0x15(%ebp)
   0x5655617c <main.c>: mov    -0x1ffc(%eax),%ecx
   0x56556182 <main.c>: mov    %ecx,-0x11(%ebp)
   0x56556185 <main.c>: mov    -0x1ff8(%eax),%ecx
   0x5655618b <main.c>: mov    %ecx,-0xd(%ebp)
   0x5655618e <main.c>: mov    -0x1ff4(%eax),%al
   0x56556194 <main.c>: mov    %al,-0x9(%ebp)
   0x56556197 <main.c>: cmpl   $0x2,0x8(%ebp)
....
   
```

maintenant on a trouvé la partie du code qui compare les deux chaines de caractères, et on peut trouver la chaine de caractères recherchée en analysant le code assembleur de cette partie du code.

J'ai trouvé le  code mais un problème se pose, la chiane compaée est stockée en mémoire à l'aide d'une addresse relative, mais cette addresse est calculée d`uatre qui n'est pas facile à trouver.

Encore la solution viendra de l'outil `gdb`, qui nous permettra de trouver la valeur de cette addresse relative, et ainsi trouver la chaine de caractères recherchée.

#### Etape 3.2.5: Trouver las chaines en fonction de l'addresse relative
deplacons le pointeur juste avant l'affectation des variables en foncton de cette addresse.

```bash
break *0x56556173
continue
```

#### Etape 3.2.6: Afficher la valeur de l'addresse relative

```bash
(gdb) x/4bx $eax-0x2000
0x56557000:     0x0b    0x31    0x76    0x23
(gdb) x/4bx $eax-0x1ffc
0x56557004:     0x21    0x62    0x23    0x66
(gdb) x/4bx $eax-0x1ff8
0x56557008:     0x2b    0x2f    0x72    0x34
(gdb) x/4bx $eax-0x1ff4
0x5655700c:     0x00    0x4e    0x65    0x65
```

#### Etape 3.2.7: Trouver la chaine de caractères recherchée

```
0x5655620c <main.c>: mov    0xc(%ebp),%eax
```

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





