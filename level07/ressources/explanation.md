si on ouvre l'executable sur ghidra:

![ghidra](./ghidra.png)

`./level07` renvoi juste ce que contient la variable environnement LOGNAME. On a juste a la modifier avec getflag et le tour est joue!

```bash
level07@SnowCrash:~$ export LOGNAME=\`getflag\`
level07@SnowCrash:~$ ./level07
```
