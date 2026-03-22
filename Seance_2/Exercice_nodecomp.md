Exercice nodecomp

Mot de passe à trouve : "Ha22Y_H4rR1s0n"

Méthode de résolution : 
Dans cette exercice on va utliser des logiciels reverse pour nous aider dans la résolution de ce challenge, et on va suivre les étapes suivantes:

## Etape 1: Analyse de l'entete du fichier
grace à l'outils `file` on peut avoir des informations, domme quoi si le binaire a été strippé ou pas.
```bash
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse_static$ file nodecomp
nodecomp: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=03a784cffe3b96bc328fadee409e8e4e56746aa8, for GNU/Linux 3.2.0, stripped
```
On peut voir que le binaire est un exécutable ELF 64 bits, statiquement lié, et qu'il a été strippé (c'est à dire que les symboles de debug ont été supprimés).

### Le point d'entrée du programme
grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.
```bash
   Entry point address:               0x401550
```

Maintenant on a une idéé globale sur le binaare, on va pousser l'étude plus loin pour trouver la clef avec les outils de reverse engineering.

## Etape 2: Analyse du code avec Ghidra

Avant de commencer l'analyse du code, il vient important de comprendre le fonctionnement du programme en l'éxécutant.

En lançant le programme on peut constater trois comportements:

- 1) nombre d'arguments, `argc`, $argc ≠ 2$.
- 2) $ argc = 2 $ mais l'argument n'est pas valide.
- 3) $ argc = 2 $ et l'argument est valide.

### Etape 2.1 Recherche du `main` avec ghidra.

En ouvrant ghidra, on peut voir un onglet `entry` qui semble etre le point d'entrée ded notre programme.

```
 004b5c94, 004b7e18(*)  
        00401550 f3 0f 1e fa     ENDBR64
        00401554 31 ed           XOR        EBP,EBP
        00401556 49 89 d1        MOV        R9,RDX
        00401559 5e              POP        RSI
        0040155a 48 89 e2        MOV        RDX,RSP
        0040155d 48 83 e4 f0     AND        RSP,-0x10
        00401561 50              PUSH       RAX
        00401562 54              PUSH       RSP=>local_10
        00401563 45 31 c0        XOR        R8D,R8D
        00401566 31 c9           XOR        ECX,ECX
        00401568 48 c7 c7        MOV        RDI,main
                 30 1a 40 00
        0040156f 67 e8 9b        CALL       FUN_00403410                                     undefined FUN_00403410()
                 1e 00 00

```

L'adresse de `entry` correspond bien l'adresse que lon avait expecté, le premier qui est l'adresse de la fonction main que j'ai modifié volontairement sa signature pourque sa lecture soit plus lisible.

### Etape 2.2 Analyse de la fonction main

Code décompilé:
```
int main(int argc,char **argv)

{
  int iVar1;
  undefined8 uVar2;
  
  if (argc != 2) {
    FUN_0040c270("Usage: %s <clef>\n",*argv);
    FUN_0040b740(0);
  }
  FUN_00401680();
  FUN_004017a0();
  uVar2 = FUN_00401c60(argv[1]);
  FUN_00401680();
  iVar1 = FUN_00401fc0(&DAT_0049903c,uVar2);
  if (iVar1 == 0) {
    FUN_0040c270("Well done\n");
  }
  else {
    FUN_0040c270("Try again\n");
  }
  FUN_00420a80(uVar2);
  return 0;
} 
```
#### Etape 2.2.1 Analyse de la fonction `FUN_00401680()`

```

        00401680 55              PUSH       RBP
        00401681 48 89 e5        MOV        RBP,RSP
        00401684 cc              INT3
        00401685 5d              POP        RBP
        00401686 c3              RET
        00401687 66              ??         66h    f
        00401688 0f              ??         0Fh
        00401689 1f              ??         1Fh
```

Cette fonction c'est une obfuscation, elle ne fait rien le reste de code après le `RET` ne sera jamais éxécuté.

#### Etape 2.2.2 Analyse de la fonction `FUN_004017a0()`

##### 2.2.2.1 Description

La fonction `FUN_004017a0` effectue une **vérification d’intégrité du binaire en cours d’exécution**.
Elle lit les premiers octets du fichier exécutable (`/proc/self/exe`) afin de vérifier s'il s'agit bien d'un fichier au format ELF (Executable and Linkable Format).

## 2.2.2.2 Fonctionnement 

### 1. Ouverture du binaire courant

```c
local_c = FUN_00450030("/proc/self/exe",0);
```

Ouvre le fichier représentant l'exécutable courant via `/proc/self/exe` qui est un lien symbolique vers le programme en cours d'exécution.
 `FUN_00450030` agit probablement comme un `open()`.


```c
FUN_00450160(local_c,&local_10,4);
```

Ici il lit les 4 premiers octets du fichier, ces octets correspondent au magic number ELF.

##### 2.2.2.3 Vérification du header ELF

```c
if ((((local_10 != '\x7f') || (local_f != 'E')) || (local_e != 'L')) || (local_d != 'F'))
```

Un fichier ELF valide commence toujours par :

```
0x7F 'E' 'L' 'F'
```
 Si les octets ne correspondent pas alors le fichier n’est pas reconnu comme ELF valide.

### 4. Réaction en cas d’échec

```c
_SUB_00000000 = &DAT_00499024;
```

 Si la vérification échoue alors une varaible global (`_SUB_00000000`) est modifiée.
Il est redirigé vers une autre zone mémoire (`DAT_00499024`).
Cela peut servir à :
  * Modifier le comportement du programme
  * Désactiver certaines fonctionnalités

##### Conclusion

Cette fonction est une **vérification d’intégrité simple mais classique** :

* Elle confirme que le programme est bien un ELF valide

Son impact est juste au niveau de la protection, cela ne change pas le fonctionnnement normal du programme.
---


#### Etape 2.2.3 Analyse de la fonction `FUN_00401c60()`

Cette fonction prend en argument une chaîne de caractère, et retourne une valeur qui sera utilisée par la suite par la fonction `FUN_00401fc0`.

En analysant le code de cette fonction, j'ai apreçu que cette une fonction obfuscée qui appelle plusieurs fonction imbriquées les unes dans les autres, tout enrespectant la signature de la fonction,

```
(char *) fonction(char *arg);
```
Mais à chaque appel d'une fonction imbriquée, l'argument à l'apppel de la fonction est modifié, voyons cette suite d'appels de fonctions imbriquées:

```
argv[1]-10+1-9+2-8+3-7+4-6+5-5+6-4+7-3+8-2+9-1+10
```

Le dernier argument de la dernière fonction imbriquée est `argv[1]`, et après une suite de modifications de cet argument, on trouve que le dernier argument de la dernière fonction imbriquée est `argv[1]+10-10` qui est égal à `argv[1]`.

Donc cette obfuscation ne fait que de modifier l'argument à chaque appel de fonction imbriquée, mais à la fin le résultat est le même que l'argument d'entrée, c'est à dire `argv[1]`.

De ce fait nous allons nous intérésser à l'appel de la fonction 
`char * FUN_00401910(char *arg)` qui est la dernière fonction imbriquée, et qui prend en argument `argv[1]`.

##### Etape 2.2.3.1 Analyse de la fonction `FUN_00401910()`

```asm
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined FUN_00401910()
             undefined         <UNASSIGNED>   <RETURN>
             undefined8        Stack[-0x18]:8 local_18                                XREF[2]:     00401919(W), 
                                                                                                   00401926(R)  
             undefined4        Stack[-0x1c]:4 local_1c                                XREF[1]:     00401923(W)  
                             FUN_00401910                                    XREF[3]:     FUN_00401bc0:00401bd0(c), 
                                                                                          004b5cdc, 004b7f20(*)  
        00401910 55              PUSH       RBP
        00401911 48 89 e5        MOV        RBP,RSP
        00401914 53              PUSH       RBX
        00401915 48 83 ec 28     SUB        RSP,0x28
        00401919 48 89 7d f0     MOV        qword ptr [RBP + local_18],RDI
        0040191d 8b 05 3e        MOV        EAX,dword ptr [DAT_00499061]                     = CCFF2342h
                 77 09 00
        00401923 89 45 ec        MOV        dword ptr [RBP + local_1c],EAX
        00401926 48 8b 4d f0     MOV        RCX,qword ptr [RBP + local_18]
        0040192a 48 89 f9        MOV        RCX,RDI
        0040192d 48 8d 1d        LEA        RBX,[LAB_0040193a]
                 06 00 00 00
        00401934 53              PUSH       RBX=>LAB_0040193a
        00401935 e9 96 f7        JMP        thunk_FUN_00422e70                               undefined * thunk_FUN_00422e70(v
                 ff ff
                             -- Flow Override: CALL_RETURN (CALL_TERMINATOR)
                             LAB_0040193a                                    XREF[2]:     FUN_00401910:0040192d(*), 
                                                                                          FUN_00401910:00401934(*)  
        0040193a 89 c1           MOV        ECX,EAX
        0040193c 89 4d d4        MOV        dword ptr [RBP + -0x2c],ECX
        0040193f 8b 45 d4        MOV        EAX,dword ptr [RBP + -0x2c]
        00401942 89 45 e8        MOV        dword ptr [RBP + -0x18],EAX
        00401945 48 63 7d e8     MOVSXD     RDI,dword ptr [RBP + -0x18]
        00401949 48 c1 e7 00     SHL        RDI,0x0
        0040194d e8 ee ed        CALL       FUN_00420740                                     undefined FUN_00420740()
                 01 00
        00401952 48 89 45 d8     MOV        qword ptr [RBP + -0x28],RAX
        00401956 c7 45 e4        MOV        dword ptr [RBP + -0x1c],0x0
                 00 00 00 00
                             LAB_0040195d                                    XREF[1]:     004019c0(j)  
        0040195d 48 8b 45 f0     MOV        RAX,qword ptr [RBP + -0x10]
        00401961 0f be 00        MOVSX      EAX,byte ptr [RAX]
        00401964 83 f8 00        CMP        EAX,0x0
        00401967 0f 84 58        JZ         LAB_004019c5
                 00 00 00
        0040196d 48 8b 45 f0     MOV        RAX,qword ptr [RBP + -0x10]
        00401971 0f be 00        MOVSX      EAX,byte ptr [RAX]
        00401974 48 63 4d e4     MOVSXD     RCX,dword ptr [RBP + -0x1c]
        00401978 0f be 4c        MOVSX      ECX,byte ptr [RBP + RCX*0x1 + -0x14]
                 0d ec
        0040197d 31 c8           XOR        EAX,ECX
        0040197f 88 c1           MOV        CL,AL
        00401981 48 8b 45 d8     MOV        RAX,qword ptr [RBP + -0x28]
        00401985 88 08           MOV        byte ptr [RAX],CL
        00401987 48 8d 3d        LEA        RDI,[FUN_00401690]
                 02 fd ff ff
        0040198e 31 c0           XOR        EAX,EAX
        00401990 89 c6           MOV        ESI,EAX
        00401992 e8 19 34        CALL       FUN_00454db0                                     undefined FUN_00454db0()
                 05 00
        00401997 48 8b 45 f0     MOV        RAX,qword ptr [RBP + -0x10]
        0040199b 48 83 c0 01     ADD        RAX,0x1
        0040199f 48 89 45 f0     MOV        qword ptr [RBP + -0x10],RAX
        004019a3 48 8b 45 d8     MOV        RAX,qword ptr [RBP + -0x28]
        004019a7 48 83 c0 01     ADD        RAX,0x1
        004019ab 48 89 45 d8     MOV        qword ptr [RBP + -0x28],RAX
        004019af 8b 45 e4        MOV        EAX,dword ptr [RBP + -0x1c]
        004019b2 83 c0 01        ADD        EAX,0x1
        004019b5 b9 04 00        MOV        ECX,0x4
                 00 00
        004019ba 99              CDQ
        004019bb f7 f9           IDIV       ECX
        004019bd 89 55 e4        MOV        dword ptr [RBP + -0x1c],EDX
        004019c0 e9 98 ff        JMP        LAB_0040195d
                 ff ff
                             LAB_004019c5                                    XREF[1]:     00401967(j)  
        004019c5 48 8b 45 d8     MOV        RAX,qword ptr [RBP + -0x28]
        004019c9 0f be 40 ff     MOVSX      EAX,byte ptr [RAX + -0x1]
        004019cd 89 45 d0        MOV        dword ptr [RBP + -0x30],EAX
        004019d0 8b 45 e4        MOV        EAX,dword ptr [RBP + -0x1c]
        004019d3 83 c0 03        ADD        EAX,0x3
        004019d6 b9 04 00        MOV        ECX,0x4
                 00 00
        004019db 99              CDQ
        004019dc f7 f9           IDIV       ECX
        004019de 8b 45 d0        MOV        EAX,dword ptr [RBP + -0x30]
        004019e1 48 63 ca        MOVSXD     RCX,EDX
        004019e4 0f be 4c        MOVSX      ECX,byte ptr [RBP + RCX*0x1 + -0x14]
                 0d ec
        004019e9 31 c8           XOR        EAX,ECX
        004019eb 48 8b 4d f0     MOV        RCX,qword ptr [RBP + -0x10]
        004019ef 0f be 49 ff     MOVSX      ECX,byte ptr [RCX + -0x1]
        004019f3 29 c8           SUB        EAX,ECX
        004019f5 83 c0 01        ADD        EAX,0x1
        004019f8 66 89 c1        MOV        CX,AX
        004019fb 48 8d 05        LEA        RAX,[LAB_00401a0c]
                 0a 00 00 00
        00401a02 66 f7 e9        IMUL       CX
        00401a05 ff e0           JMP        RAX
        00401a07 e9              ??         E9h
        00401a08 fc              ??         FCh
        00401a09 ff              ??         FFh
        00401a0a ff              ??         FFh
        00401a0b ff              ??         FFh
                             LAB_00401a0c                                    XREF[1]:     004019fb(*)  
        00401a0c 48 8b 45 d8     MOV        RAX,qword ptr [RBP + -0x28]
        00401a10 48 63 55 e8     MOVSXD     RDX,dword ptr [RBP + -0x18]
        00401a14 31 c9           XOR        ECX,ECX
        00401a16 48 29 d1        SUB        RCX,RDX
        00401a19 48 01 c8        ADD        RAX,RCX
        00401a1c 48 83 c4 28     ADD        RSP,0x28
        00401a20 5b              POP        RBX
        00401a21 5d              POP        RBP
        00401a22 c3              RET

```

##### 1. Initialisation
```
MOV [RBP+local_18], RDI
MOV EAX, [DAT_00499061]
MOV [RBP+local_1c], EAX
```
param_1 (RDI) est sauvegardé en tant que local_18, et une constante globale est chargée dans EAX et sauvegardé en tant que local_1c.
Chargement d’une constante globale :
DAT_00499061 = 0xCCFF2342

##### 2. Appel d’une fonction externe

```
LEA RBX, [LAB_0040193a]
PUSH RBX
JMP thunk_FUN_00422e70
```

Ici on ajoute l'adresse de LAB_0040193a qui sera utilisée comme adresse de retour pour la fonction FUN_00422e70, et on saute à cette fonction.

En analysant cette fonction via ces références croisées, j'ai pu constater que c'est une fonction système qui retourne une adresse fonction, ce qui nous aide pas pour l'instant dans notre quete de trouver une fonction qui envoie une adresse en fonction de `RDI`.

on va supposer que cette fonction est une obfuscation.

##### 3. Allocation de mémoire
```
CALL FUN_00420740
MOV [RBP-0x28], RAX
```

EN analysant le code de la fonction en ligne , on a pu constaté que la fonction malloc, qui alloue de la mémoire et retourne un pointeur vers cette mémoire allouée, est appelée à l'adresse `0x00420740`, et le résultat de cette fonction est sauvegardé dans `RAX` et ensuite sauvegardé dans la variable locale `local_28`.

#### 4 Analise de la boucle principale


##### 4.1 Initialisation de la boucle

```
        00401956 c7 45 e4        MOV        dword ptr [RBP + -0x1c],0x0
                 00 00 00 00
```

##### 4.2 Condition de la boucle

Condition de terminer la boucle:
```
                             LAB_0040195d                                    XREF[1]:     004019c0(j)  
        0040195d 48 8b 45 f0     MOV        RAX,qword ptr [RBP + -0x10]
        00401961 0f be 00        MOVSX      EAX,byte ptr [RAX]
        00401964 83 f8 00        CMP        EAX,0x0
        00401967 0f 84 58        JZ         LAB_004019c5
```

Si on regarde bienle code, on peut constater que la condition de terminer la boucle est lorsque le caractère pointé par `RAX` est égal à 0, c'est à dire lorsque le caractère nul est atteint de la chaine était passé en argument de la fonction, soit `argv[1]`.
Car on veut partir de `[RBP + -0x10]` vers `[RBP + -0x18]`. 
Ce qui est equivalent à parcourir la chaîne jusqu’au caractère nul.

##### 4.3 Corps de la boucle

```C
  while (**(char **)(unaff_RBP + -0x10) != '\0') {
    **(byte **)(unaff_RBP + -0x28) =
         **(byte **)(unaff_RBP + -0x10) ^
         *(byte *)(unaff_RBP + -0x14 + (long)*(int *)(unaff_RBP + -0x1c));
    FUN_00454db0(FUN_00401690,0);
    *(long *)(unaff_RBP + -0x10) = *(long *)(unaff_RBP + -0x10) + 1;
    *(long *)(unaff_RBP + -0x28) = *(long *)(unaff_RBP + -0x28) + 1;
    *(int *)(unaff_RBP + -0x1c) = (*(int *)(unaff_RBP + -0x1c) + 1) % 4;
  }
```

En regardant le code décompilé de la boucle, on peut voir ici l'alogorithme de chiffrement qui est utilisé pour chiffrer la clé passée en argument, et c'est à partir de cet algorithme que je vais trouver la clé.

```
**(byte **)(unaff_RBP + -0x28) = **(byte **)(unaff_RBP + -0x10) ^ *(byte *)(unaff_RBP + -0x14 + (long)*(int *)(unaff_RBP + -0x1c));
```

Que l'on pourrait résumer par la boucle suivante:
```C
for (int i = 0; argv[1][i] != '\0'; i++) {
    encrypted_key[i] = argv[1][i] ^ key[i % 4];
}
```

Et la clé de chiffrement qui a été est celle ci:
```c
cle_chiffre = [0x42, 0x23, 0xFF, 0xCC];
```

##### 5. Après la boucle

```
                             LAB_00401a0c                                    XREF[1]:     004019fb(*)  
        00401a0c 48 8b 45 d8     MOV        RAX,qword ptr [RBP + -0x28]
        00401a10 48 63 55 e8     MOVSXD     RDX,dword ptr [RBP + -0x18]
        00401a14 31 c9           XOR        ECX,ECX
        00401a16 48 29 d1        SUB        RCX,RDX
        00401a19 48 01 c8        ADD        RAX,RCX
        00401a1c 48 83 c4 28     ADD        RSP,0x28
        00401a20 5b              POP        RBX
        00401a21 5d              POP        RBP
        00401a22 c3              RET
```

Ici il y a beaucoup d'obfuscation, comme :
```
        00401a14 31 c9           XOR        ECX,ECX
        00401a16 48 29 d1        SUB        RCX,RDX
```
Qui auront des valeurs nulles, et qui ne feront rien, et d'autres instructions qui font que le résultat final de cette fonction est de retourner l'adresse de la clé chiffrée.

au final l'adresse de la clé chiffrée est celle qui est pointé par `RAX` à la fin de la fonction, et cette adresse est celle de `local_28` qui contient la clé chiffrée.

## Etape 3: Trouver le code chiffré
Maintenant revenons à la fonction `main`, après l'appel de la fonction `FUN_00401910` qui retourne l'adresse de la clé chiffrée, cette adresse est sauvegardé dans `uVar2`, et ensuite cette adresse est passé en argument à la fonction `FUN_00401fc0` qui compare la clé chiffrée avec une clé de référence, et si les deux clés sont égales le programme affiche "Well done", sinon il affiche "Try again".

### Analyse de la fonction `FUN_00401fc0` qui compare la clé chiffrée avec la clé de référence.

Après analyse de cette fonction, j'ai pu percevoir encore de l'obfuscation en permuttant les arguments de la fonction avec plusieurs fonctions imbriquées.

Mais j'ai pu constaté que le nombre de fois que `arg1` et `arg2` ont été permuté est un nombre pair, ce qui fait que à la fin le résultat de la fonction est de comparer `arg1` avec `arg2`, et c'est à partir de cette comparaison que je vais trouver la clé de référence.

Donc nous allons analyser la dernière fonction imbriquée qui compare les deux clés, et qui retourne 0 si les deux clés sont égales, et un nombre différent de 0 sinon.

### Analyse de la fonction de comparaison des clés `FUN_00401820(param_1,param_2)`

En analysant le code de ctte fonction elle contient beaucoup d'obfuscation, mais le résultat final de cette fonction est de comparer les deux clés, et de retourner 0 si les deux clés sont égales, et un nombre différent de 0 sinon.

### Recherche de la clé de référence
Maintenant que nous avons trouvé l'algorithme de chiffrement, et que nous avons trouvé que la fonction de comparaison des clés compare la clé chiffrée avec une clé de référence, il nous reste à trouver la clé de référence pour pouvoir trouver la clé d'entrée qui est le mot de passe à trouver.

La clé de référence est stockée à l'adresse `&DAT_0049903c`.
```
                             DAT_0049903c                                    XREF[1]:     main:00401a90(*)  
        0049903c 0a              ??         0Ah
        0049903d 42              ??         42h    B
        0049903e cd              ??         CDh
        0049903f fe              ??         FEh
        00499040 1b              ??         1Bh
        00499041 7c              ??         7Ch    |
        00499042 b7              ??         B7h
        00499043 f8              ??         F8h
        00499044 30              ??         30h    0
        00499045 71              ??         71h    q
        00499046 ce              ??         CEh
        00499047 bf              ??         BFh
        00499048 72              ??         72h    r
        00499049 4d              ??         4Dh    M
        0049904a 00              ??         00h
```

En inversant l'algorithme de chiffrement, et en utilisant la clé de chiffrement que nous avons trouvé précédement, on trouve que la clé de référence est "Ha22Y_H4rR1s0n".


