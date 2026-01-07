en jouant avec l'executable `ex09` on s'apercoit qu'il print la string et il incremente la valeur ascii par rapport au nombre de caractere

```./level09 aaaaaaaaaaaaaaaaaaaaaaaaaa
abcdefghijklmnopqrstuvwxyz```

pour decoder le token on a juste a appliquer le meme algo mais inverse, au lieu de +1 on -1

# fonction pour appliquer l'alog sur le token

vim `/tmp/main.c`
```#include <stdio.h>

int main(int ac, char **av) {
    int i = 0;
    char c;
    
    while (av[1][i] != 0) {
        c = av[1][i];
        printf("%c", (c - i));
        i++;
    }
    printf("\n");
    return 0;
}```

```cd /tmp```
```cc main.c```
```cd /tmp```
```./a.out cat `~/token'```

on utilise le token pour se auth:

`su flag09`
`getflag`
