# Explications

## Premiere piste
Il n'y a quasiment aucun indice sur ce niveau , aucun fichier dans le repertoire..
J'ai commence par regarder `l'env` et il y avait une variable `MAIL` je me suis alors dis que il y avait un serveur quelque part et que on allait essayer comme dans d'autres niveaux de le faire executer un truc mais non.. il y a bien des processus qui ecoutent sur le port 4646 et 4747 (apache) mais ca semble etre des fausses pistes apres quelque essais.

A cours d'idee on se met a regarder sur le discord et apparement plusieurs personnes sont dans le meme cas, donc nos indices sont venus de discords en nous disant que c'etait dans la continuite du `level13` et en lien avec `getflag` donc il faut surement reverse engineer `getflag` lui meme (toujours pas convaincu que ce soit la bonne solution mais si elle fonctionne alors on pouvait l'utiliser sur tout les niveaux ?).

## Reverse engineer de `getflag`

Si on utilise `gdb` sur `getflag` et que on run le binaire on va voir ceci :

```sh
level14@SnowCrash:~$ gdb /bin/getflag
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /bin/getflag...(no debugging symbols found)...done.
(gdb) run
Starting program: /bin/getflag
You should not reverse this
[Inferior 1 (process 3807) exited with code 01]
```

`You should not reverse this`, le progamme est donc au courant que on essaye de le reverse engineer soit c'est vraiment un warning pour ne pas nous autoriser a le faire, pourtant j'ai envie de faire tout l'inverse.

Le programme est au courant que l'on utilise un debugger car il utilise `ptrace` dans son code et que ce dernier echoue si un `ptrace` est deja en cours (c'est le cas lorsque que on lance le programme avec gdb).

Voici une meilleure explication (https://bases-hacking.org/ptrace.html):

```txt
I°) La méthode ptrace

Explication de la méthode
   Elle est assez simple d'utilisation : sous les systèmes UNIX, un appel système nommé ptrace sert à débogguer les éxécutables (les tracer, les modifier, poser des points d'arrêt, etc..). Il va de soi que cet appel système est utilisé par tous les debuggers sous Unix. L'appel ptrace est déclaré dans la librairie système sys/ptrace.h. Ptrace a la particularité qu'il ne peut être utilisé qu'une seule fois sur un même éxécutable en simultané. En fait, ptrace retourne 0 si le lancement a réussi et -1 si il y a déjà un ptrace en cours sur l'éxécutable.
Il paraît donc naturel qu'il suffit d'appeller ptrace dans le programme : s'il réussit, tout est normal, s'il échoue, il y a un débuggage en cours sur l'éxécutable et cela ne présage rien de bon.
```

Pour etre sur et certains que le programme utilise `ptrace` on peut faire ceci dans gdb :
```sh
level14@SnowCrash:~$ gdb /bin/getflag
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /bin/getflag...(no debugging symbols found)...done.
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x08048444  _init
0x08048490  strdup
0x08048490  strdup@plt
0x080484a0  __stack_chk_fail
0x080484a0  __stack_chk_fail@plt
0x080484b0  getuid
0x080484b0  getuid@plt
0x080484c0  fwrite
0x080484c0  fwrite@plt
0x080484d0  getenv
0x080484d0  getenv@plt
0x080484e0  puts
0x080484e0  puts@plt
0x080484f0  __gmon_start__
0x080484f0  __gmon_start__@plt
0x08048500  open
0x08048500  open@plt
0x08048510  __libc_start_main
0x08048510  __libc_start_main@plt
0x08048520  fputc
0x08048520  fputc@plt
0x08048530  fputs
0x08048530  fputs@plt
0x08048540  ptrace
0x08048540  ptrace@plt
0x08048550  _start
0x08048580  __do_global_dtors_aux
0x080485e0  frame_dummy
0x08048604  ft_des
0x0804871c  syscall_open
0x0804874c  syscall_gets
0x080487be  afterSubstr
0x08048843  isLib
0x08048946  main
0x08048ed0  __libc_csu_init
0x08048f40  __libc_csu_fini
0x08048f42  __i686.get_pc_thunk.bx
0x08048f50  __do_global_ctors_aux
0x08048f7c  _fini
```

Voici le peu de choses a connnaitre sur l'assembleur avant de continuer :

- `info functions` -> commande gdb qui liste les fonctions connues dans le binaire
- `EAX` -> valeur de retour d’une fonction
- `info registers eax` -> commande gdb qui affiche la valeur actuelle du registre `eax`
- `finish` -> laisse la fonction s'executer puis stop juste apres

### Etape 1 : Contourner `l'anti-debug`

On sait que `ptrace` retourne `0` si l'appel reussi et `-1` si il echoue et quitte le programme avec le message `You should not reverse this`. Notre premiere etape va donc etre de faire croire que programme que l'appel a reussi pour pouvoir continuer.

On va s'y prendre comme ceci :

```sh
break ptrace
run
finish
set $eax = 0
continue
```

qui va nous donner :

```sh
(gdb) break ptrace
Breakpoint 1 at 0x8048540
(gdb) run
Starting program: /bin/getflag

Breakpoint 1, 0xb7f146d0 in ptrace () from /lib/i386-linux-gnu/libc.so.6
(gdb) finish
Run till exit from #0  0xb7f146d0 in ptrace () from /lib/i386-linux-gnu/libc.so.6
0x0804898e in main ()
(gdb) set $eax = 0
(gdb) continue
Continuing.
Check flag.Here is your token :
Nope there is no token here for you sorry. Try again :)
[Inferior 1 (process 4066) exited normally]
```

Parfait on voit que on a reussi a contourner `ptrace` en modifiant sa valeur de retour a 0.
Le flag ne nous est cependant pas donne et c'est normal, si on regarde les functions qui etait affiches avec `info functions` on remarque qu'il y avait `getuid` celle ci est responsable de recuper notre `id` et par la suite verifier si nous avons le droit de recuperer le flag il nous faudrait donc mentir sur notre `id` en se faisant passer pour `flag14`.

### Etape 2 : Se faire passer pour `flag14`

Le chose a faire est de connaitre l'id de `flag14` , pour ce faire allons voir le fichier `/etc/passwd`

```sh
flag14:x:3014:3014::/home/flag/flag14:/bin/bash
```

Parfait l'id de `flag14` est donc `3014`.

Reprenons maintenant le debug de `getflag` , il nous faut refaire le contournement de `ptrace` mais ce coup ci on va ajouter un deuxieme breakpoint : `break getuid`, et nous allons modifier le retour de `getuid` on va y mettre `3014` :

```sh
level14@SnowCrash:~$ gdb /bin/getflag
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /bin/getflag...(no debugging symbols found)...done.
(gdb) break ptrace
Breakpoint 1 at 0x8048540
(gdb) break getuid
Breakpoint 2 at 0x80484b0
(gdb) run
Starting program: /bin/getflag

Breakpoint 1, 0xb7f146d0 in ptrace () from /lib/i386-linux-gnu/libc.so.6
(gdb) finish
Run till exit from #0  0xb7f146d0 in ptrace () from /lib/i386-linux-gnu/libc.so.6
0x0804898e in main ()
(gdb) info registers eax
eax            0xffffffff	-1  # <-- Ici on voit bien que ptrace retournait -1
(gdb) set $eax = 0
(gdb) continue
Continuing.

Breakpoint 2, 0xb7ee4cc0 in getuid () from /lib/i386-linux-gnu/libc.so.6
(gdb) finish
Run till exit from #0  0xb7ee4cc0 in getuid () from /lib/i386-linux-gnu/libc.so.6
0x08048b02 in main ()
(gdb) info registers eax
eax            0x7de	2014 # <-- Ici on voit bien que getuid retournait notre id de level14
(gdb) set $eax = 3014
(gdb) continue
Continuing.
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 4377) exited normally]
```
