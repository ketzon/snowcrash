# Explication

En faisait `ls` on observe un fichier `level04.pl`, il s'agit d'un script Perl.

```sh
level04@SnowCrash:~$ ls -l
total 4
-rwsr-sr-x 1 flag04 level04 152 Mar  5  2016 level04.pl
```

`-rwsr-sr-x` nous indique que le script se lancera avec droits de `flag04` et du groupe `level04`.

Observons le script :
```sh
level04@SnowCrash:~$ cat ./level04.pl
#!/usr/bin/perl
# localhost:4747
use CGI qw{param};
print "Content-type: text/html\n\n";
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
x(param("x"));
```

- `localhost:4747` -> simple indication pour nous aiguiller a communiquer a cette addresse.
- `use CGI qw{param};` -> import de param de la librairie CGI, c'est utilise pour traitrer des parametres de requetes webs.
- `print "Content-type: text/html\n\n";` -> Header qui sera dans la reponse du serveur.
- `x(param("x"));` -> Appel la fonction `x` en passant comme parametre le parametre`"x"` de la requete HTTP.

Maintenant le plus important la fonction `x` :
```sh
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
```
- `$y = $_[0];` -> creer une variable `y` prenant comme valeur le premier parametre passe a la fonction.
- ``print `echo $y 2>&1`;`` -> `echo $y 2>$1` va s'executer dans un shell et `print` va simplement afficher le retour de la sortie du shell.

## Exploiter la faille

Ce script s'execute avec les droits de `flag04` lorsque l'on envoit une requete a "http://localhost:4747/?x=$var" et on controle totalement ce parametre qui sera utilise pour etre execute dans le shell via `` `echo $y 2>&1`;``. Il nous faut alors envoyer un payload pour faire en sorte que notre parametre soit interpreter comme une commande shell.

Ici `echo getflag 2>&1` va etre executer donc evidemment pas bon.
```sh
level04@SnowCrash:~$ curl http://localhost:4747/?x=getflag
getflag
```

Mais apres plusieurs essais dans un shell si on entoure un commande de backticks celle ci va etre executee et c'est la sortie de la commande qui va remplacer la commande initiale. Donc si je fait ceci dans un shell `` `getflag` `` la commande `getflag` va se faire et la sortie deviendra mon nouvel input. Si j'arrive donc a envoyer `` `getflag` `` dans mon parametre `x` la commande execute par le script sera `` `echo `getflag` 2>&1` ``, `` `getflag` `` va s'executer avec les droits de `flag04` et retourner `ne2searoevaevoem4ov4ar8ap` , cela deviendra donc `` `echo ne2searoevaevoem4ov4ar8ap 2>&1` ``. Pour finir le script va `print` le retour de ce shell et donc tout simplement le flag. Le tout nous est renvoye en reponse HTTP.

Pour pouvoir envoyer des backticks dans ma requete je dois utiliser des `single quotes` sinon le shell va interpreter `getflag` dans mon shell avant de faire la requete.

```sh
level04@SnowCrash:~$ curl http://localhost:4747/?x='`getflag`'
Check flag.Here is your token : ne2searoevaevoem4ov4ar8ap
```
