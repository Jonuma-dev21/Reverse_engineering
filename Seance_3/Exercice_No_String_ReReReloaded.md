Exercice : No String ReReReloaded

Mot de passe à trouve : "PHil1p_k*_diCK"

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

```C
int main(int argc,char **argv)

{
  char *pcVar1;
  int iVar2;
  size_t sVar3;
  char local_c8 [128];
  char local_48 [32];
  undefined *local_28;
  undefined *local_20;
  char **local_18;
  int local_10;
  undefined4 local_c;
  
  local_c = 0;
  local_20 = &DAT_5555557d4004;
  local_28 = &DAT_5555557d400d;
  local_48[0] = -9;
  local_48[1] = 'q';
  local_48[2] = '\r';
  local_48[3] = 'I';
  local_48[4] = -0x48;
  local_48[5] = '\a';
  local_48[6] = 'e';
  local_48[7] = 'O';
  local_48[8] = -0x27;
  local_48[9] = -0x5b;
  local_48[10] = ']';
  local_48[0xb] = 'f';
  local_48[0xc] = -0xc;
  local_48[0xd] = ')';
  local_48[0xe] = -0x5b;
  local_48[0xf] = '\x11';
  local_48[0x10] = 0;
  local_18 = argv;
  local_10 = argc;
  if (argc != 2) {
    printf("Usage: %s <password>\n",*argv);
                    /* WARNING: Subroutine does not return */
    exit(-1);
  }
  FUN_5555555ccca0(0,"legacy");
  pcVar1 = local_18[1];
  sVar3 = strlen(local_18[1]);
  iVar2 = FUN_5555555b8250(pcVar1,sVar3 & 0xffffffff,local_20,local_28,local_c8);
  local_c8[iVar2] = '\0';
  iVar2 = strcmp(local_c8,local_48);
```

Ce qui nous ineresse pour l'instant c'est la fonction qui modife la variable local_c8, en focntion de l'entrée de l'utilisateur, et c'est à partir de cette fonction que je vais pouvoir trouver la chaine de caractères du mot de passe.

On peut voir voir claiarement que c'est une fonction de chiffrement, et que le résultat du chiffrement est comparé à une chaine de caractères local_48, et c'est à partir de cette comparaison que je vais pouvoir trouver la chaine de caractères du mot de passe.

### Etape 5 Trouver cette fonction de chiffrement.

Avant d'analyser cette vérifions quelques pistes donner dans l'appelant de la fonction de chiffrement, on peut voir que la fonction de chiffrement utilise une clé de chiffrement qui est stockée dans les variables local_20 et local_28, et c'est à partir de cette clé de chiffrement que je vais pouvoir trouver la chaine de caractères du mot de passe.


Normalement une fonction de chiffrement prend une clé de chiffrement, la seconde est surement un IV qui est utilisé pour le chiffrement, et la troisième variable est surement le buffer de sortie du chiffrement, et c'est à partir de cette variable que je vais pouvoir trouver la chaine de caractères du mot de passe.

avec IV on est sur que c'est un chiffrement symétrique, et avec la taille de la clé de chiffrement qui est de 16 bytes, on peut supposer que c'est un chiffrement AES-128-CBC, et c'est à partir de cette supposition que je vais pouvoir trouver la chaine de caractères du mot de passe.

Mais il ya deux remarques important que l'ai fait l'appel à une fonction qui prend en entrée `legacy` comme argument et la taille de la clé de chiffrement qui est de 8 bytes et 8 bytes pour l'IV, ce qui correspond surement  à un chiffrement du passé, `DES`, et c'est à partir de cette supposition que je vais pouvoir trouver la chaine de caractères du mot de passe.

clé de chiffrement :
```
                             DAT_5555557d4004                                XREF[3]:     main:5555555b8149(*), 
                                                                                          main:5555555b8150(*), 
                                                                                          main:5555555b81dc(*)  
    5555557d4004 0d              ??         0Dh
    5555557d4005 24              ??         24h    $
    5555557d4006 2b              ??         2Bh    +
    5555557d4007 ad              ??         ADh
    5555557d4008 68              ??         68h    h
    5555557d4009 56              ??         56h    V
    5555557d400a 6a              ??         6Ah    j
    5555557d400b e6              ??         E6h
    5555557d400c 00              ??         00h

```

IV :
```bash
                             DAT_5555557d400d                                XREF[3]:     main:5555555b8154(*), 
                                                                                          main:5555555b815b(*), 
                                                                                          main:5555555b81e0(*)  
    5555557d400d 3c              ??         3Ch    <
    5555557d400e 7d              ??         7Dh    }
    5555557d400f 21              ??         21h    !
    5555557d4010 1e              ??         1Eh
    5555557d4011 53              ??         53h    S
    5555557d4012 d6              ??         D6h
    5555557d4013 a1              ??         A1h
    5555557d4014 42              ??         42h    B
    5555557d4015 00              ??         00h
```


On est pas sur que c'est un chiffrement DES, mais c'est une piste à explorer, et c'est à partir de cette piste que je vais pouvoir trouver la chaine de caractères du mot de passe.

#### Cherchons le cyphertext utilisé dans la fonction de comparaison local_48.
Avec gdb on refait les memes étapes que pour l'exercice précédent, pour trouver avec que données sont comparés le résultat du chiffrement.

```bash
Breakpoint 1, 0x00005555555b8210 in ?? ()
(gdb) x/s $rsi
0x7fffffffd980: "\367q\rI\270\aeO٥]f\364)\245\021"
(gdb) x/16xb $rsi
0x7fffffffd980: 0xf7    0x71    0x0d    0x49    0xb8    0x07    0x65    0x4f
0x7fffffffd988: 0xd9    0xa5    0x5d    0x66    0xf4    0x29    0xa5    0x11
(gdb) x/17xb $rsi
0x7fffffffd980: 0xf7    0x71    0x0d    0x49    0xb8    0x07    0x65    0x4f
0x7fffffffd988: 0xd9    0xa5    0x5d    0x66    0xf4    0x29    0xa5    0x11
0x7fffffffd990: 0x00
```

Maintenant on a:
clef de chiffrement : 0d 24 2b ad 68 56 6a e6
IV : 3c 7d 21 1e 53 d6 a1 42
Cyphertext : f7 71 0d 49 b8 07 65 4f d9 a5 5d 66 f4 29 a5 11

#### Étape 6 : Trouver la chaine de caractères du mot de passe avec un outil de déchiffrement en ligne
Nous allons tenté de chiffrer tout cela avec openssl pour trouver la chaine de caractères du mot de passe, mais avant de faire cela, il faut convertir la clé de chiffrement, l'IV et le cyphertext en base64 pour pouvoir les utiliser avec openssl.

Preparation du cyphertext pour openssl : en convertissant le cyphertext en binaire avec xxd, et en l'enregistrant dans un fichier enc.bin.

```bash
 echo f7710d49b807654fd9a55d66f429a511 | xxd -r -p > enc.bin
```

Dechiffrement du cyphertext avec openssl :
```bash
openssl enc -des-cbc -d -provider legacy -K 0d242bad68566ae6 -iv 3c7d211e53d6a142 -in enc.bin
```

resultat du déchiffrement : "PHil1p_k*_diCK"



