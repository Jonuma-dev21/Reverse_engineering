Exercice : No String Reloaded

Mot de passe à trouve : "N.*K.*j3m1s1n"

Méthode de résolution : 

Dans cet exercice nous allons devoir faire une étude de la chaine de manière dynamique pour le password.

### Étape 1 : Exécution du programme

En €xecutant le programme on peut voir que celui ci attend une et une seule entrée de la part de l'utilisateur, et après l'entrée du mot de passe le programme affiche un message d'erreur "Try again" et se termine.

### Étape 2 : Analyse du code avec ltrace

ltrace est un outil de traçage de bibliothèques partagées qui permet de suivre les appels de fonctions et les interactions avec les bibliothèques partagées utilisées par un programme.

```
jonumabee@JonumaISI:~/Documents/S2/SDS/dynamic$ ltrace ./nsr maman
memcpy(0x7ffd3ff34ea0, "v\352\335\310\323\307O\211\233\326\375\004\037\027\2161\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 128) = 0x7ffd3ff34ea0
EVP_CIPHER_CTX_new(0x7ffd3ff34ea0, 16, 0x5e7a38873004, 0x5e7a38873060)                              = 0x5e7a3cc1b2a0
EVP_aes_256_cbc(0, 0, 0, 0)                                                                         = 0x7ebcdc2be380
EVP_DecryptInit_ex(0x5e7a3cc1b2a0, 0x7ebcdc2be380, 0, 0x5e7a38873004)                               = 1
EVP_DecryptUpdate(0x5e7a3cc1b2a0, 0x7ffd3ff34e10, 0x7ffd3ff34dbc, 0x7ffd3ff34ea0)                   = 1
EVP_DecryptFinal_ex(0x5e7a3cc1b2a0, 0x7ffd3ff34e10, 0x7ffd3ff34dbc, 1)                              = 1
EVP_CIPHER_CTX_free(0x5e7a3cc1b2a0, 0, 0, 1)                                                        = 10
strcmp("maman", "N.*K.*j3m1s1n")                                                                    = 31
printf("Try again\n"Try again
)                                                                               = 10
+++ exited (status 0) +++
```

En analysant le code avec ltrace on peut voir que le programme utilise la bibliothèque OpenSSL pour effectuer une opération de déchiffrement AES-256-CBC, et après le déchiffrement il compare l'entrée de l'utilisateur avec une chaîne de caractères "N.*K.*j3m1s1n" à l'aide de la fonction strcmp.

Mais ce qui nous intéresse c'est la comparaison de l'entrée de l'utilisateur avec la chaîne de caractères "N.*K.*j3m1s1n", et c'est à partir de cette comparaison que nous allons tenter de trouver le mot de passe.

### Étape 3 : Trouver le mot de passe en testant la chaine vue dans ltrace

Après le test il s'avère que le mot de passe "N.*K.*j3m1s1n" est le bon mot de passe pour ce challenge, et en l'entrant dans le programme on peut voir que celui ci affiche un message de succès "Well done" et se termine.