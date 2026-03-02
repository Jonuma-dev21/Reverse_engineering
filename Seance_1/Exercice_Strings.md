Exercice NOM-CHALLENGE

Mot de passe à trouve : Ors0n Sco77 C4rd

Méthode de résolution :

Etape 1 : informations sur l'entete du fichier
Avant do commencer il faut que j'aie un minimum d'informations sur le fichier pour savoir dans quelle section se trouve son code source.

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

Etape 2: Affichage du code assembleur de cette section 

commande 
```bash
objdump -d string .text
```

Ou on peut voir la comparaison des deux chaines ici

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