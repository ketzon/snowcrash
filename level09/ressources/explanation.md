# Level09 — Décodage du token

En jouant avec l’exécutable `level09`, on s’aperçoit qu’il affiche la chaîne passée en argument, mais **en incrémentant la valeur ASCII de chaque caractère selon sa position**.

Exemple :

```bash
./level09 aaaaaaaaaaaaaaaaaaaaaaaaaa
```

Output :

```
abcdefghijklmnopqrstuvwxyz
```

👉 Donc le caractère à la position `i` est transformé en :  
```
c' = c + i
```

---

## 🔓 Objectif : décoder le token

Pour décoder le token, il suffit donc **d'appliquer l’algorithme inverse** :

```
c' = c - i
```

---

## 🧠 Programme pour décoder

On crée un programme en C qui applique l’inversion :

Créer `/tmp/main.c`

```c
#include <stdio.h>

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
}
```

Compiler :

```bash
cd /tmp
cc main.c
```

Puis exécuter sur le token :

```bash
./a.out `cat ~/token`
```

Cela affiche **le token décodé** 🎉

---

## 🔑 Utilisation du token

```bash
su flag09
getflag
```

Flag obtenu 😎
