# Explication

Un `ls` nous permet de voir deux fichiers dans le home :

```sh
level06@SnowCrash:~$ ls -l
total 12
-rwsr-x---+ 1 flag06 level06 7503 Aug 30  2015 level06
-rwxr-x---  1 flag06 level06  356 Mar  5  2016 level06.php
```

Un executable et du code php , tout deux s'excutant avec les droits de `flag06`. On comprend assez rapidement que le code de `level06.php` semble etre celui de l'executable `level06`.

Observons un peu le code du php ci dessous :

```php
…/snow-crash-perso/level06 ❯ cat level06.php
#!/usr/bin/php
<?php
function y($m)
{
    $m = preg_replace("/\./", " x ", $m);
    $m = preg_replace("/@/", " y", $m);

    return $m;
}

function x($y, $z)
{
    $a = file_get_contents($y);
    $a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a);
    $a = preg_replace("/\[/", "(", $a);
    $a = preg_replace("/\]/", ")", $a);

    return $a;
}

$r = x($argv[1], $argv[2]);
print $r;
?>
```

C'est un programme qui prend 2 arguments (1 seul est suffisant cependant l'autre n'est pas utilise dans le code), le premier est un fichier qui sera ouvert par `$a = file_get_contents($y)`. Le contenu de ce fichier sera ensuite passer dans une fonction `preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a)` et c'est de cette fonction que va venir la faille.

`preg_replace($pattern, $replacement, $string);` permet de chercher quelque chose dans une string et de remplacer celle ci par une autre.
Ici le `$pattern` est defini dans le code deja present mais il utilise `/e` qui est une ancienne fonctionnalite qui n'etait pas safe et qui va permettre l'interpretation de ce qui se trouve dans `$replacement` en code php. Ici `$string` represente le contenu de notre fichier, c'est d'ici que viendra notre payload.

## Comment exploiter ?

Quand notre fichier sera ouvert par l'application son contenu va passer par le premier `preg_replace()`, la fonction va chercher le `$pattern` dans notre fichier et va vouloir le remplacer par ce qu'il y a dans `$replacement`.

Explication regex :

- `"/(\[x (.*)\])/e"` -> `(\[x (.*)\])` tout ce qui est la dedans fait partie de la premiere capture, si on enleve les caracteres d'echappement le pattern ressemble a `[x ]`. Le deuxieme groupe de capture est `(.*)`, il signifie n'importe quel caractere. Le deuxieme groupe se trouve a l'interieur du premier ce qui donne pour patern complet `[x <input>]`.
- `"y(\"\\2\")"` -> Utiliser ce qui est dans la deuxieme capture et l'inserer dans la string, executer la fonction `y` grace a `/e` dans le `$pattern`

Il nous faut donc ecrire un payload qui sera valider par le `$pattern` afin que la deuxieme capture se retrouve dans la partie `"y(\"\\2\")"` qui sera evalue en tant que code php

Apres pleins de tentatives de casser les strings dans ``"y(\"\\2\")"`` pour y executer d'autre commandes il y avait finalement plus simple.

Il est possible d'utiliser des backticks pour executer des commandes shell et recuperer la sortie et de l'assigner a une variable (https://www.php.net/manual/en/language.operators.execution.php), pour que ce payload marche il faut egalement qu'il soit dans une une string double quotes (c'est le cas) et que le code soit evaluer en tant que code php (c'est le cas grace a `/e`). Donc si je peux faire `getflag` et pour l'assigner a une variable je fait `` ${`getflag`}`` , il me reste a le mettre dans mon payload : ``[x ${`getflag`}]``

On peut creer un fichier contenant le payload dans `/tmp` et le donner au programme ensuite

```sh
level06@SnowCrash:~$ ./level06 /tmp/hack.txt
No entry for terminal type "xterm-kitty";
using dumb terminal settings.
PHP Notice:  Undefined variable: Check flag.Here is your token : wiok45aaoguiboiki2tuin6ub
 in /home/user/level06/level06.php(4) : regexp code on line 1
 ```

 Le retour de la commande `getflag` est stockee dans la variable qui elle meme utiliser en tant que paramatre de ``y("${`getflag`}");`` , la fonction y s'execute et la programme se termine en printant notre sortie.
