Exercice Tictactoe

Mot de passe à trouve : "FranK her83R7"

Méthode de résolution : 
Dans ces exercice on va utliser des logiciels reverse pour nous aider dans la résolution de ce challenge, et on va suivre les étapes suivantes:

Comme Cutter+radare2 et Ghidra sont des logiciels de reverse engineering, ils nous permettent d'analyser le code binaire d'un programme pour comprendre son fonctionnement.

## Etape 1: Analyse de l'entete du fichier
grace à l'outils `type` on peut avoir des informations, domme quoi si le binaire a été strippé ou pas.
```bash
[jonumabee@jonuma reverse_static]$ file tictactoe 
tictactoe: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=a65c1626e2f974e29c44fbb65e8997770b28576c, for GNU/Linux 3.2.0, stripped
```
On peut voir que le binaire est un exécutable ELF 64 bits, statiquement lié, et qu'il a été strippé (c'est à dire que les symboles de debug ont été supprimés).

### Le point d'entrée du programme
grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.
```bash
  Adresse du point d'entrée:         0x401960
```
Maintenant on a une idéé globale sur le binaare, on va pousser l'étude plus loin pour trouver le mot de passe avec les outils de reverse engineering.

## Etape 2: Analyse du code avec Cutter+radare2

Avant de commencer l'analyse du code, il vient important de comprendre le fonctionnement du programme en l'éxécutant. 

Analyse des branches d'éxécution du programme à travers le menu du jeu, et en essayant de gagner une partie de tic tac toe pour voir ce qui se passe après la victoire.

### Play
Après l'éxécution du programme on peut constater qu'après avoir gagné une partie de tic tac toe, le programme affiche un message de victoire et nous demande d'enresgistrer notre nom pour le sauvegarder.

### Scoreboard

Ce menu affiche le nom du dernier gagnant qui doit etre un invité, mais a mon analyse de l'éxécution du programme et de ce qui est demqndé dans le challenge, il existe trois cas d'affichage dans la section du scoreboard:

Et ce sera à partir de ces comportements que je vais essayer de trouver le nom de l'invité spécial.

- Cas 1: il n'y pas eu de gagnant avant et le programme affiche "Still waiting for the first winner..."

- Cas 2: il y a eu un gagnant avant mais ce n'est pas l'invité spécial, et le programme affiche "The last winner was: <nom du gagnant>"

- Cas 3: il y a eu un gagnant avant et c'est l'invité spécial, et le programme affichera un message spécial pour ce gagnant, et c'est à partir de ce message que je vais trouver le nom de l'invité spécial.

Maintenant reprenons l'analyse du code avec Cutter+radare2 pour trouver le nom de l'invité spécial.

Avec l'analyse précédente du code, l'idée sera de retrouver cette affichage  du message spécial , normalement il doit y avoir une condition le précédent gagnat doit être égal à l'invité spécial pour que le programme affiche ce message spécial, et c'est à partir de cette condition que je vais trouver le nom de l'invité spécial.

### Recherche du message spécial dans le code
En analysant le code avec Cutter+radare2, j'ai trouvé une section du Strings le message suivant 

```
"Last game has been won by...\n    OUR SPECIAL GUEST! ;)"
```

Logiquement ce message doit etre affiché à la place du message "The last winner was: <nom du gagnant>" dans le cas ou le précédent gagnant est égal à l'invité spécial via une condition de comparaison.

### Recherche de la condition de comparaison

Nous allons tenté de remonter le code pour trouver les références à ce message spécial, et trouver la condition de comparaison qui doit être quelque chose comme "if (last_winner == special_guest) { print message_special }".


En remontant le code, j'ai trouvé la référnece à ce message spécial a été invoqué.

```
0x0040297f      cmp     dword [0x0052b92c], 0
0x00402986      jne     0x4029aa
0x0040298c      mov     rdi, qword [rbp - 8]
0x00402990      lea     rsi, [str.Well_done_to_our_last_winner:__s] ; 0x4e805d
0x00402997      lea     rdx, [0x0052b910]
0x0040299e      mov     al, 0
0x004029a0      call    fcn.00406cd0 ; fcn.00406cd0
0x004029a5      jmp     0x4029bc
0x004029aa      mov     rdi, qword [rbp - 8]
0x004029ae      lea     rsi, [str.Last_game_has_been_won_by..._____OUR_SPECIAL_GUEST] ; 0x4e807e
0x004029b5      mov     al, 0
0x004029b7      call    fcn.00406cd0 ; fcn.00406cd0
0x004029bc      mov     rax, 0x52bcf0
0x004029c3      mov     rdi, qword [rax]
0x004029c6      call    fcn.00407250 ; fcn.00407250
```

En analysant le grapphe de ce code j'ai trouvé la condition de comparaison qui dit quand sauté à l'adresse `0x4029aa` pour afficher le message spécial, et c'est à partir de cette condition que je vais trouver le nom de l'invité spécial.

```
0x0040297f      cmp     dword [0x0052b92c], 0
0x00402986      jne     0x4029aa
```

Cette condition de comparaison compare la valeur à l'adresse `0x0052b92c` avec 0, et si cette valeur n'est pas égale à 0, le programme saute à l'adresse `0x4029aa` pour afficher le message spécial.

Cherchons à quelle moment cette adresse `0x0052b92c` est modifiée pour trouver le nom de l'invité spécial.

### Recherche de la modification de l'adresse `0x0052b92c`
En faisant une recherche de référence croisée pour l'adresse `0x0052b92c`, 

```
0x00401f44      sub     rax, rcx
0x00401f47      sub     rax, 9
0x00401f4b      cmp     rax, 3     ; 3
0x00401f4f      je      0x401f5a
0x00401f55      jmp     0x401f64
0x00401f5a      mov     dword [0x0052b92c], 1
```
j'ai trouvé que cette adresse est modifiée à l'adresse `0x401f5a` pour être égal à 1, et cette modification se fait après une à suite de comparaisons dans une fopnction que je vais analyser pour trouver le nom de l'invité spécial.

### Etape 3: Analyse de la fonction qui modifie l'adresse `0x0052b92c`



