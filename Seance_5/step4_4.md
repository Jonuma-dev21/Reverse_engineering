Exercice : Step 4/4 Botnet

Mot de passe à trouve :  vstarcam2015

Méthode de résolution :

Objectif : `What is the login associated to the password "20150602"?`

Dans la liste des procesus affichée par le plugin `linux.pslist.PsList`, j'ai pu remarquer que plusieurs processus `su` ont été lancés , ce qui correspond surment à des tentatives de changement d'utilisateur.

Pour trouver le login associé au mot de passe "20150602", nous allons analyser les processus `su` pour trouver des informations sur les tentatives de changement d'utilisateur, et c'est à partir de ces informations que nous allons pouvoir trouver le login associé au mot de passe "20150602".

En poursuivant ce piste je n'ai rien trouvé de piste exploitable pour trouver le login associé au mot de passe "20150602", et je pense que le mot de passe "20150602" est associé à un login qui n'est pas utilisé dans les processus `su` affichés par le plugin `linux.pslist.PsList`, et c'est pour cela que je n'ai pas trouvé de piste exploitable pour trouver le login associé au mot de passe "20150602".

### Analyse du processus /bin/systemd de pid 7295 avec ghidra

En ouvrant ghidra, j'ai pu remarquer que le block de code du entry_point n'était pas interprété comme étant du code.

affichage du block de code du entry_point dans ghidra :

```asm
                             DAT_00400194                                    XREF[1]:     00400018(*)  
        00400194 31              ??         31h    1
        00400195 ed              ??         EDh
        00400196 49              ??         49h    I
        00400197 89              ??         89h
        00400198 d1              ??         D1h
        00400199 5e              ??         5Eh    ^
        0040019a 48              ??         48h    H
        0040019b 89              ??         89h
        0040019c e2              ??         E2h
        0040019d 48              ??         48h    H
        0040019e 83              ??         83h
        0040019f e4              ??         E4h
        004001a0 f0              ??         F0h
        004001a1 50              ??         50h    P
        004001a2 54              ??         54h    T
        004001a3 48              ??         48h    H
        004001a4 c7              ??         C7h
        004001a5 c7              ??         C7h
        004001a6 70              ??         70h    p
        004001a7 39              ??         39h    9
        004001a8 40              ??         40h    @
        004001a9 00              ??         00h
        004001aa 48              ??         48h    H
        004001ab c7              ??         C7h
        004001ac c1              ??         C1h
        004001ad e8              ??         E8h
        004001ae 00              ??         00h
        004001af 40              ??         40h    @
        004001b0 00              ??         00h
        004001b1 49              ??         49h    I
        004001b2 c7              ??         C7h
        004001b3 c0              ??         C0h
        004001b4 06              ??         06h
        004001b5 d1              ??         D1h
        004001b6 40              ??         40h    @
        004001b7 00              ??         00h
        004001b8 e8              ??         E8h
        004001b9 39              ??         39h    9
        004001ba b7              ??         B7h
        004001bb 00              ??         00h
        004001bc 00              ??         00h
        004001bd f4              ??         F4h
        004001be 90              ??         90h
        004001bf 90              ??         90h
```

Pour pallier à ce problème, j'si fait en sorte que ghidra interprète ce block de code comme étant du code en faisant `disassemble`.

Resultat:

```asm
                             LAB_00400194                                    XREF[1]:     00400018(*)  
        00400194 31 ed           XOR        EBP,EBP
        00400196 49 89 d1        MOV        R9,RDX
        00400199 5e              POP        RSI
        0040019a 48 89 e2        MOV        RDX,RSP
        0040019d 48 83 e4 f0     AND        RSP,-0x10
        004001a1 50              PUSH       RAX
        004001a2 54              PUSH       RSP
        004001a3 48 c7 c7        MOV        RDI,FUN_00403970
                 70 39 40 00
        004001aa 48 c7 c1        MOV        RCX,LAB_004000e8
                 e8 00 40 00
        004001b1 49 c7 c0        MOV        R8,LAB_0040d106
                 06 d1 40 00
        004001b8 e8 39 b7        CALL       FUN_0040b8f6                                     undefined FUN_0040b8f6()
                 00 00
        004001bd f4              HLT
        004001be 90              NOP
        004001bf 90              NOP

```

### Analyse de la fonction main `FUN_00403970`
'
Le code du main etait très difficile à analyser, et j'ai du faire plusieurs hypothèses pour comprendre le code, et c'est à partir de ces hypothèses que j'ai pu trouver des informations sur le login associé au mot de passe "20150602".

J'ai du utliser une technique qui vise à trouver la chaine rechercher elle meme en parcourant ce code au niveau des donnnées, on pouvait voir en clair quelques chaines de caractères mais il n'y avait aucune piste exploitable pour trouver le login associé au mot de passe "20150602", et c'est pour cela que j'ai du faire cette technique pour trouver la chaine rechercher elle meme.


Mais j'ai pu remarque une section bizarre

```
                             DAT_0040d5e8                                    XREF[4]:     FUN_00402ac0:00402c93(*), 
                                                                                          FUN_00402ac0:00402cf5(*), 
                                                                                          FUN_00402ac0:00402d42(*), 
                                                                                          0050fc90(*)  
        0040d5e8 2f              ??         2Fh    /
        0040d5e9 78              ??         78h    x
        0040d5ea 33              ??         33h    3
        0040d5eb 38              ??         38h    8
        0040d5ec 2f              ??         2Fh    /
        0040d5ed 78              ??         78h    x
        0040d5ee 46              ??         46h    F
        0040d5ef 4a              ??         4Ah    J
        0040d5f0 2f              ??         2Fh    /
        0040d5f1 78              ??         78h    x
        0040d5f2 39              ??         39h    9
        0040d5f3 33              ??         33h    3
        0040d5f4 2f              ??         2Fh    /
        0040d5f5 78              ??         78h    x
        0040d5f6 49              ??         49h    I
        0040d5f7 44              ??         44h    D
        0040d5f8 2f              ??         2Fh    /
        0040d5f9 78              ??         78h    x
        0040d5fa 39              ??         39h    9
        0040d5fb 41              ??         41h    A
        0040d5fc 2f              ??         2Fh    /
        0040d5fd 78              ??         78h    x
        0040d5fe 33              ??         33h    3
        0040d5ff 38              ??         38h    8
        0040d600 2f              ??         2Fh    /
        0040d601 78              ??         78h    x
        0040d602 46              ??         46h    F
        0040d603 4a              ??         4Ah    J
        0040d604 2f              ??         2Fh    /
        0040d605 78              ??         78h    x

```

On dirait des données encodées.

Maintenant l'idée c'est de trouver la fonction qui fait cette encodage, et de l'appliquer sur ces données pour trouver des informations sur le login associé au mot de passe "20150602".

### Examinons la fonction `FUN_00402ac0` qui fait référence à cette section de données bizarre.

Dans cette fonction, j'ai pu remarquer que les données de la section bizarre sont utilisées dans une boucle qui fait des opérations de XOR et de décalage sur ces données, ce qui correspond à une technique d'encodage/décodage.

```

/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */

void FUN_00404650(void)

{
  undefined4 uVar1;
  uint uVar2;
  
  uVar1 = _DAT_00510140;
  uVar2 = _DAT_00510138 << 0xb ^ _DAT_00510138;
  _DAT_00510138 = _DAT_0051013c;
  _DAT_00510140 = _DAT_00510144;
  _DAT_0051013c = uVar1;
  _DAT_00510144 = uVar2 >> 8 ^ uVar2 ^ _DAT_00510144 ^ _DAT_00510144 >> 0x13;
  return;
}

```

mais étant données que la Clé est dynamique je vais tenté de reconstituér la Clé j'arrive à trouver dans cette zone de mémoire toutes les caractères imprimables et qui contient "20150602".

```python
from pathlib import Path

ELF_FILE = "final.elf"
SECTION_OFFSET = 0x0d5e8
SECTION_SIZE   = 0x2b30

TARGET = b"20150602"

blob = Path(ELF_FILE).read_bytes()
data = blob[SECTION_OFFSET:SECTION_OFFSET + SECTION_SIZE]

print("[+] Brute force XOR key to find:", TARGET.decode())

for key in range(256):
    decrypted = bytes(b ^ key for b in data)

    if TARGET in decrypted:
        print("\n[!!!] KEY FOUND =", hex(key))
        
        
        idx = decrypted.find(TARGET)
        start = max(0, idx - 40)
        end = idx + 40

        print("\n[Context]:")
        print(decrypted[start:end])

        break
else:
    print("[-] No key found")
```

En executant ce code j'ai trouvé la valeur `0xea`

Et j'ai trouvé le login comme afficher ici.

```
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ python3 search_key.py 
[+] Brute force XOR key to find: 20150602

[!!!] KEY FOUND = 0xea

[Context]:
b'xx.\xeaZte521\xeaanko\xeazsun1188\xeaxc3511\xeaxmhdpic\xea20150602\xeavstarcam2015\xea12345678\xeaadmin\xea123'
```

