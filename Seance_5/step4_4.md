Exercice : Step 4/4 Botnet

Mot de passe à trouve :  

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

