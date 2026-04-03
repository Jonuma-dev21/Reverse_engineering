Exercice : Step 2/4 malware

Mot de passe à trouve : "MBEDTLS,RSA_AES_CBC,2048,128"

Méthode de résolution : 

### Step 2/4 : Give, in this order, upper-case and coma-separated, the following elements:

- name of the crypto lib.
- name of asymmetric algorithm, name of symmetric algorithm and mode (separated by underscore).
- asymmetric key length.
- symmetric key length.


#### Étape 2.1 : Analyse du binaire avec ghidra

Suite à l'étape 1/4, nous avons trouvé que le binaire file1 est le malware qui chiffre les données du système, et que file2 contient une clé publique qui est surement utilisée pour chiffrer les données du système.

Une fonction, `encrypt_directory`, nous a particulièrement intéressé dans le binaire file1, c'est la fonction qui est utilisée pour chiffrer les données du système, et c'est à partir de cette fonction que nous allons pouvoir trouver les éléments demandés dans l'exercice.

#### Étape 2.2 : Trouver les éléments demandés dans l'exercice à partir de la fonction `encrypt_directory`

An analysant le code de la fonction `encrypt_directory`, j'ai remarqué la présence d'une autre focntion qui est `encrypt_file`, cette fonction est utilisée pour chiffrer les fichiers du système, et c'est à partir de cette fonction que nous allons pouvoir trouver les éléments demandés dans l'exercice.

#### Étape 2.3 : Trouver les éléments demandés dans l'exercice à partir de la fonction `encrypt_file`

```C
aes_encrypt(local_68,0x10,__ptr,local_58,local_78);
```

Avec cette signature de la fonction `aes_encrypt`, nous pouvons identifier quelques éléments demandés :

Regadons d'abord la taille des arguments de la fonction `aes_encrypt` :

```
  undefined1 local_78 [16];
  undefined1 local_68 [16];
```
- La taille de `local_78` est de 16 octets, ce qui correspond à une clé de 128 bits pour l'algorithme AES.
- La taille de `local_68` est également de 16 octets, ce qui correspond au `IV` (Initialization Vector) utilisé dans les modes de chiffrement symétrique.

Ensuite, le nom de la fonction `aes_encrypt` nous indique que l'algorithme de chiffrement symétrique utilisé est AES de taille 128 bits,  et que le mode de chiffrement est probablement CBC (Cipher Block Chaining) ou CTR (Counter), car ces modes nécessitent un IV.

Maintenant, pour trouver le nom de la librairie nous allons analyser la fonction `loadRSA` 

```C
mbedtls_ctr_drbg_init(ctr_drbg);
```

La fonction `mbedtls_ctr_drbg_init` est une fonction de la librairie `mbed TLS`, donc le nom de la librairie utilisée pour le chiffrement est `MBEDTLS`.

Mainteant, vu le nom de la fonction fonction on aune une piste sur le nom du chiffrement asymétrique utilisé, qui est RSA, et pour trouver la taille de la clé asymétrique utilisée, nous allos directement analyser le fichier file2 qui contient la clé publique utilisée pour le chiffrement asymétrique, et avec la commande `openssl` on peut trouver la taille de la clé asymétrique utilisée.

```bash
openssl pkey -pubin -in file2 -text -noout
```

resultat de la commande :

```
Public-Key: (2048 bit)
Modulus:
    00:b5:7d:60:a1:bc:dc:5b:d7:68:35:e6:70:99:dc:
    49:aa:6b:12:37:e1:b0:98:99:24:f5:56:c1:26:37:
    65:04:7f:11:61:d4:eb:1a:ed:34:bb:b2:6f:6a:45:
    2e:a9:76:16:da:18:35:18:87:51:df:be:cd:4e:91:
    5f:89:a7:5a:c2:ba:7f:b7:36:c8:e0:00:80:0b:af:
    d0:28:dc:e1:a0:8d:54:8d:d5:3d:52:f4:a9:3f:28:
    49:24:a0:49:d5:4f:3f:33:fd:62:a8:10:ea:07:26:
    e3:99:ce:7c:5c:4f:99:cc:b1:12:2e:9b:21:b6:a2:
    49:28:b5:cd:7d:45:a4:85:2e:ed:d0:6a:11:27:70:
    0b:39:b2:43:e4:63:78:65:94:4b:25:cb:ff:1a:fe:
    8e:7b:1b:e7:ce:cf:3a:f8:29:f4:ce:fc:12:90:68:
    01:31:f4:c5:0f:ff:c8:fc:8e:97:e3:51:e1:f1:a4:
    f6:3b:02:83:c5:ca:72:42:11:a6:b1:20:30:7e:44:
    24:a7:d0:2c:e4:e7:e7:55:a4:57:5b:6e:f0:43:94:
    2c:33:e8:3c:17:d3:b2:de:01:18:d5:4a:c8:e9:b1:
    18:f0:f5:b7:fa:8e:ae:87:c9:de:dc:a1:19:ec:4f:
    1d:8f:b5:c4:39:24:29:e6:e5:10:a3:95:30:77:3f:
    a6:13
Exponent: 65537 (0x10001)
```

- Le nom de la librairie utilisée pour le chiffrement est `MBEDTLS`.
- Le nom de l'algorithme de chiffrement asymétrique utilisé est `RSA`, le nom de l'algorithme de chiffrement symétrique utilisé est `AES`, et le mode de chiffrement est probablement `CBC`.
- La taille de la clé asymétrique utilisée est de 2048 bits.
- La taille de la clé symétrique utilisée est de 128 bits.  

d'ou la réponse finale : `MBEDTLS,RSA_AES_CBC,2048,128`



