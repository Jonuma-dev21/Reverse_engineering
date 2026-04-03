Exercice : Step 1/4 malware

Mot de passe à trouve : "/home,/root,/var/lib/mysql,/var/www,/etc/nginx,/etc/apache2,/var/log"

Méthode de résolution : 


Contexte du piratage  :

Un attaquant à réussi à s'introduire dans un système et a laissé un malware qui chiffre les données du système precisémenr des repertoires. Mais on aréussi à récupérer quelques traces laissées par l'attaqunt.

- file1
- file2
- dump.img

### Étape 1 : Analyse de file1

```bash
file file1
file1: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
```

grace à l'outil file on peut voir que file1 est un binaire ELF, c'est surement le malware qui éxécute le chiffrement des données du système, que nous allons analyser par la suite.

### Étape 2 : Analyse de file2

```bash
jonubee@jonubee-VirtualBox:~/Documents/S2/SDS/malware$ file file2
file2: ASCII text
jonubee@jonubee-VirtualBox:~/Documents/S2/SDS/malware$ cat file2
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtX1gobzcW9doNeZwmdxJ
qmsSN+GwmJkk9VbBJjdlBH8RYdTrGu00u7JvakUuqXYW2hg1GIdR377NTpFfiada
wrp/tzbI4ACAC6/QKNzhoI1UjdU9UvSpPyhJJKBJ1U8/M/1iqBDqBybjmc58XE+Z
zLESLpshtqJJKLXNfUWkhS7t0GoRJ3ALObJD5GN4ZZRLJcv/Gv6Oexvnzs86+Cn0
zvwSkGgBMfTFD//I/I6X41Hh8aT2OwKDxcpyQhGmsSAwfkQkp9As5OfnVaRXW27w
Q5QsM+g8F9Oy3gEY1UrI6bEY8PW3+o6uh8ne3KEZ7E8dj7XEOSQp5uUQo5Uwdz+m
EwIDAQAB
-----END PUBLIC KEY-----
jonubee@jonubee-VirtualBox:~/Documents/S2/SDS/malware$
```
file2 est un fichier texte qui contient une clé publique, cette clé publique est surement utilisée pour chiffrer les données du système, et c'est à partir de cette clé publique que nous allons pouvoir trouver la chaine de caractères du mot de passe.

### Étape 3 : Analyse de dump.img

```bash
jonubee@jonubee-VirtualBox:~/Documents/S2/SDS/malware$ file dump.img 
dump.img: DOS/MBR boot sector, extended partition table (last)
```

dump.img est une image de disque qui contient une table de partition étendue, cette table de partition étendue contient des informations sur les partitions du disque, cette image est l'image de la machine qui a été attaquée.

### Step 1/4 : objectif `Give the list (coma separated) of folders targeted by the ransomware.`

Pour trouver la liste des dossiers ciblés par le ransomware, nous allons analyser le binaire file1 avec ghidra pour trouver les codes qui sont utilisées pour chiffrer les données du système, et c'est à partir de ces codes que nous allons pouvoir trouver la liste des dossiers ciblés par le ransomware.

#### Étape 3.1 : Reverse du binaire avec ghidra

Nous allons directemant dans le main du binaire ou on a trouver ces deux lignes.

```C
  arg_1 = argv[1];
  iVar2 = strcmp(arg_1,"encrypt");
```
attend une entrée de la part de l'utilisateur, et si cette entrée est "encrypt" alors le programme va chiffrer les données du système, sinon il va dans une branche pour déchiffrer les données du système, mais dans notre cas on va se concentrer sur la branche de chiffrement des données du système pour trouver la liste des dossiers ciblés par le ransomware.

#### Étape 3.2 : Trouver la liste des dossiers ciblés par le ransomware

```C
  if (iVar2 == 0) {
    puts("Start encrypting...");
    loadRSA(argv[2],1);
    if (3 < argc) {
      htmlPath = strdup(argv[3]);
      loadHtmlFile(htmlPath);
    }
    loadTextFile();
    createDaemon();
    lVar5 = 0;
    unlink(*argv);
    do {
      puVar1 = (undefined8 *)((long)&dirs + lVar5);
      lVar5 = lVar5 + 8;
      encrypt_directory(*puVar1);
    } while (lVar5 != 0x38);
    while (ppVar4 = getpwent(), ppVar4 != (passwd *)0x0) {
      encrypt_directory(ppVar4->pw_dir);
    }
    setpwent();
    up_encrypt(acStack_418);
    find_and_do(&DAT_0042f148);
    if ((htmlPath != (char *)0x0) && (__stream = fopen64(htmlPath,"ab+"), __stream != (FILE *)0x0))
    {
      fputs(indexHtml,__stream);
      fclose(__stream);
    }
  }
```
Cette partie du code dit beaucoup de choses, mais ce qui nous intéresse c'est la partie ou il y a la boucle do while, cette boucle itère sur une liste de dossiers qui sont stockés dans une variable dirs, et c'est à partir de cette variable dirs que nous allons pouvoir trouver la liste des dossiers ciblés par le ransomware.

```C
    lVar5 = 0;
    unlink(*argv);
    do {
      puVar1 = (undefined8 *)((long)&dirs + lVar5);
      lVar5 = lVar5 + 8;
      encrypt_directory(*puVar1);
    } while (lVar5 != 0x38);
```

On peut remarquer que `dirs` est un pointeur de pointeur de caractère, et que la boucle itère sur une liste de 7 éléments (0x38/8 = 7), donc on peut conclure que la liste des dossiers ciblés par le ransomware est composée de 7 éléments, et c'est à partir de cette liste que nous allons pouvoir trouver la liste des dossiers ciblés par le ransomware.

En analysant la variable `dirs` on trouve que les éléments de cette liste sont les suivants :

- /home
- /root
- /var/lib/mysql
- /var/www (un peu subtile sur celui ci ui est ecrit en deux parties dans le binaire)
- /etc/nginx
- /etc/apache2
- /var/log

donc de l'indice 0 à l'indice 6 de la liste `dirs` on trouve les dossiers ciblés par le ransomware, et c'est à partir de cette liste que nous allons pouvoir trouver la liste des dossiers ciblés par le ransomware.



