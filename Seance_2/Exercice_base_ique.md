Nom de l'exercice: Exercice_base_ique

mode passe: Cl1FFoRd+d+S1m4K++++

Méthode de résolution : 
Dans cette exercice on va utliser des logiciels reverse pour nous aider dans la résolution de ce challenge, et on va suivre les étapes suivantes:

## Etape 1: Analyse de l'entete du fichier
grace à l'outils `file` on peut avoir des informations, domme quoi si le binaire a été strippé ou pas.

```bash
jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/reverse_static$ file base_ique
base_ique: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=539f5bb3d2a2b7eb2cd78dd0399a66cc588f8fd2, for GNU/Linux 3.2.0, stripped
```
On peut voir que le binaire est un exécutable ELF 64 bits, statiquement lié, et qu'il a été strippé (c'est à dire que les symboles de debug ont été supprimés).

### Le point d'entrée du programme
grace à l'outils `readelf` avec l'option `-h` permet d'afficher ces informations.
```bash
Entry point address:               0x10e0
```

Maintenant on a une idéé globale sur le binaire, on va pousser l'étude plus loin pour trouver la clef avec les outils de reverse engineering.

## Etape 2: Analyse du code avec Ghidra

Avant de commencer l'analyse du code, il vient important de comprendre le fonctionnement du programme en l'éxécutant.

En lançant le programme on peut constater quatre comportements:

- 1) nombre d'arguments, `argc`, $argc ≠ 2$.
- 2) $ argc = 2 $ mais l'argument mais le format n'est pas valide.
- 3) $ argc = 2 $, le format est valide mais le mot de passe est incorrect.
- 4) $ argc = 2 $, le format est valide et le mot de passe est correct.

### Analyse du code avec Ghidra
Nous allons pour essayer de décompiler ces 4 comportements pour trouver la clef du challenge.

Voici le code C décompiler du main proposé par Ghidra:

```C
int main(int argc,char **argv)

{
  int iVar1;
  char *__s1;
  
  if (argc != 2) {
    printf("Usage: %s <password>\n",*argv);
                    
    exit(0);
  }
  __s1 = (char *)FUN_00101430(argv[1]);
  if (__s1 == (char *)0x0) {
    printf("Invalid format\n");
  }
  else {
    iVar1 = strcmp(__s1,&DAT_00102061);
    if (iVar1 == 0) {
      printf("Good password\n");
    }
    else {
      printf("Wrong password\n");
    }
    free(__s1);
  }
  return 0;
}
```

Comme on peut le voir dans le code, la fonction `FUN_00101430` est invoqué pour vérifier le format de l'argument, et si le format est valide, elle retourne une chaîne de caractères qui est comparé à une autre chaîne de caractères stocké à l'adresse `0x102061` pour vérifier si le mot de passe est correct ou pas.

### Analyse de la fonction `FUN_00101430`
En analysant la fonction `FUN_00101430` on peut constater que cette fonction 
```
  sVar5 = strlen(param_1);
  if ((sVar5 & 3) == 0) {
    uVar6 = FUN_001012e0(param_1);
    if ((uVar6 & 1) == 0) {
      local_10 = (void *)0x0;
    }
    else {
      local_10 = malloc((ulong)(((uint)sVar5 >> 2) * 3 + 1));
      local_20 = 0;
      for (local_1c = 0; local_1c < (uint)sVar5; local_1c = local_1c + 4) {
        cVar1 = FUN_001013d0((int)param_1[local_1c]);
        uVar3 = FUN_001013d0((int)param_1[local_1c + 1]);
        uVar4 = FUN_001013d0((int)param_1[local_1c + 2]);
        cVar2 = FUN_001013d0((int)param_1[local_1c + 3]);
        *(byte *)((long)local_10 + (ulong)local_20) = cVar1 << 2 | (byte)((uVar3 & 0x30) >> 4);
        *(char *)((long)local_10 + (ulong)(local_20 + 1)) =
             (char)((uVar3 & 0xf) << 4) + (char)((uVar4 & 0x3c) >> 2);
        *(char *)((long)local_10 + (ulong)(local_20 + 2)) = (char)((uVar4 & 3) << 6) + cVar2;
        local_20 = local_20 + 3;
      }
      *(undefined1 *)((long)local_10 + (ulong)local_20) = 0;
    }
  }
  else {
    local_10 = (void *)0x0;
  }
  return local_10;
```

Dans cet extrait de code on peut voir que la fonction vérifie d'abord si la longueur de la chaîne de caractères `param_1` est un multiple de 4, et si ce n'est pas le cas, elle retourne `NULL` (c'est à dire un pointeur nul).

```
  sVar5 = strlen(param_1);
  if ((sVar5 & 3) == 0)
```

Cette fonction appel une autre fonction `FUN_001012e0` , on a va vérifier le rôle de cette fonction pour comprendre le fonctionnement de `FUN_00101430`.

### Analyse de la fonction `FUN_001012e0`

```
undefined1 FUN_001012e0(char *param_1)

{
  char *local_18;
  
  local_18 = param_1;
  while( true ) {
    if (*local_18 == '\0') {
      return 1;
    }
    if ((((*local_18 < '0') || ('9' < *local_18)) && ((*local_18 < 'A' || ('Z' < *local_18)))) &&
       ((((*local_18 < 'a' || ('z' < *local_18)) && (*local_18 != '+')) &&
        ((*local_18 != '/' && (*local_18 != '=')))))) break;
    local_18 = local_18 + 1;
  }
  return 0;
}
```

Cette fonction vérifie si tous les caractères de la chaîne de caractères `param_1` sont des caractères valides pour un encodage en base64, c'est à dire des chiffres, des lettres majuscules, des lettres minuscules, ou les caractères '+' , '/' et '='.

### Conclusion
Après l'analyse de ces deux contraintes pour déterminer un format valide, on peut conclure que le mot de passe doit être une chaîne de caractères encodé en base64 et sa longueur doit être un multiple de 4, et que lorsque cette chaîne de caractères est décodé, elle doit être égale à la chaîne de caractères stocké à l'adresse `0x102061` pour que le mot de passe soit considéré comme correct.

Revenons à l'anayse de la fonction `FUN_001012e0` pour trouver la chaîne de caractères stocké à l'adresse `0x102061` pour trouver le mot de passe correct.

```      local_10 = malloc((ulong)(((uint)sVar5 >> 2) * 3 + 1));
        local_20 = 0;
```
Ici on a deux variables `local_10` et `local_20`, la première est un pointeur qui va stocker la chaîne de caractères décodé, et la deuxième est un compteur qui va être utilisé pour parcourir la chaîne de caractères encodé.

### Analyse de la boucle de décodage
```C
      for (local_1c = 0; local_1c < (uint)sVar5; local_1c = local_1c + 4) {
        cVar1 = FUN_001013d0((int)param_1[local_1c]);
        uVar3 = FUN_001013d0((int)param_1[local_1c + 1]);
        uVar4 = FUN_001013d0((int)param_1[local_1c + 2]);
        cVar2 = FUN_001013d0((int)param_1[local_1c + 3]);
        *(byte *)((long)local_10 + (ulong)local_20) = cVar1 << 2 | (byte)((uVar3 & 0x30) >> 4);
        *(char *)((long)local_10 + (ulong)(local_20 + 1)) =
             (char)((uVar3 & 0xf) << 4) + (char)((uVar4 & 0x3c) >> 2);
        *(char *)((long)local_10 + (ulong)(local_20 + 2)) = (char)((uVar4 & 3) << 6) + cVar2;
        local_20 = local_20 + 3;
      }
```

On peut voir que cette boucle parcourt la chaîne de caractères encodé par bloc de 4 caractères, et pour chaque bloc de 4 caractères, elle utilise la fonction `FUN_001013d0` pour convertir chaque caractère en une valeur que l'on va analyser, puis elle utilise ces valeurs pour calculer les 3 caractères décodé correspondants et les stocke dans la variable `local_10`.

### Analyse de la fonction `FUN_001013d0`
```C

int FUN_001013d0(char param_1)

{
  char *pcVar1;
  undefined4 local_c;
  
  pcVar1 = strchr(PTR_s_+xz1/Dbg2FL5AHmRYKG0irVIOphZ7oj4_00104078,(int)param_1);
  if (PTR_s_+xz1/Dbg2FL5AHmRYKG0irVIOphZ7oj4_00104078 < pcVar1) {
    local_c = (int)pcVar1 - (int)PTR_s_+xz1/Dbg2FL5AHmRYKG0irVIOphZ7oj4_00104078;
  }
  else {
    local_c = 0;
  }
  return local_c;
}
```

Cette fonction utilise la fonction `strchr` pour trouver la position du caractère `param_1` dans une chaîne de caractères stocké à l'adresse `0x104078`, et retourne la position de ce caractère dans la chaîne, ou 0 si le caractère n'est pas trouvé.

Cette chaine de caractères est une base particulière, elle est la suivante:
```
+xz1/Dbg2FL5AHmRYKG0irVIOphZ7oj4e9ucMyBlvWXdSawTCJtPqUk6QnsN83Ef
```

### Conclusion
En analysant le code de la fonction `FUN_00101430` et les fonctions qu'elle utilise, on peut conclure que le mot de passe doit être une chaîne de caractères encodé en base64 avec une base particulière, et que lorsque cette chaîne de caractères est décodé, elle doit être égale à la chaîne de caractères stocké à l'adresse `0x102061` pour que le mot de passe soit considéré comme correct.

Revenons à l'anayse de la fonction `FUN_001012e0` pour trouver la chaîne de caractères stocké à l'adresse `0x102061` pour trouver le mot de passe correct.

Maintenant que nous avons compris le fonctionnement de la fonction `FUN_00101430`, nous allons utiliser un script Python pour décoder la chaîne de caractères stocké à l'adresse `0x102061` en utilisant la base particulière pour trouver le mot de passe correct, avec un fonction qui permet de faire l'inverse du codage.

Mais avant tout revenoons dans le main pour trouver la chaine de caracteres en clair qui est stocké à l'adresse `0x102061` pour trouver le mot de passe correct.

```
                             DAT_00102061                                    XREF[1]:     main:0010160c(*)  
        00102061 c2              ??         C2h
        00102062 70              ??         70h    p
        00102063 c9              ??         C9h
        00102064 25              ??         25h    %
        00102065 d3              ??         D3h
        00102066 eb              ??         EBh
        00102067 02              ??         02h
        00102068 b0              ??         B0h
        00102069 2c              ??         2Ch    ,
        0010206a 0c              ??         0Ch
        0010206b e7              ??         E7h
        0010206c d1              ??         D1h
        0010206d 00              ??         00h

```

```python
mon_alphabet = "+xz1/Dbg2FL5AHmRYKG0irVIOphZ7oj4e9ucMyBlvWXdSawTCJtPqUk6QnsN83Ef"

def client_base64_encode(data: bytes) -> str:
    encoded = []

    for i in range(0, len(data), 3):
        block = data[i:i+3]

        
        b = list(block) + [0] * (3 - len(block))

        v1 = (b[0] >> 2) & 0x3F
        v2 = ((b[0] & 0x03) << 4) | ((b[1] >> 4) & 0x0F)
        v3 = ((b[1] & 0x0F) << 2) | ((b[2] >> 6) & 0x03)
        v4 = b[2] & 0x3F

        encoded.append(mon_alphabet[v1])
        encoded.append(mon_alphabet[v2])
        encoded.append(mon_alphabet[v3])
        encoded.append(mon_alphabet[v4])

    return ''.join(encoded)



donnees = bytes([0xc2, 0x70, 0xc9, 0x25, 0xd3, 0xeb, 0x02, 0xb0, 0x2c, 0x0c, 0xe7, 0xd1,0x00])

result = client_base64_encode(donnees)
print(result)

```

Le résultat de ce script est le mot de passe correct pour le challenge, qui est le suivant:
```Cl1FFoRd+d+S1m4K++++```

