# resume

Le programme level03 s’exécute avec les droits de flag03 (à cause du SUID) et lance la commande echo via system();

# piste
si on exec `ls -la` sur le binaire `level03` on obtient, `-rwsr-sr-x 1 flag03  level03 8627 Mar  5  2016 level03`. le binaire est sous set-uid flag-3. Donc tout ce qu'il execute tourne avec les droits de flag03

si on exec `strace ./level03` pour observer les appels systeme, on peut voir que le retour du binaire `Exploit me` est bien executer par un child. ce qui nous amene sur la piste que lexecutable est en realite `system("echo Exploit me")` et non pas `printf("Exploit me")` car un autre process est lance pour afficher du texte, ce n'est pas le programme principale qui ecrit directement. c'est exactement le comportement de `system()`

maintenant on sais que la commande system appel un shell pour executer echo via le $PATH, ce qui n'est pas forcement `/bin/echo`, car on peut exucuter *NOTRE FAUX* `echo`

etant donne que le programme est sous suid, si il execute notre faux `echo` on obtient une elevation de privileges car, notre code tourne avec les droits `flag03`

# exploit
notre but est de recuperer les droits `flag03` en exploitant le comportement de `system()`, system lance un shell et chercher dans le $PATH echo, a partir de la jai juste besoin de placer mon binaire avant le binaire echo.

je creer mon script dans un dossier avec tt les droit
```cd /tmp
echo "/bin/sh" > echo
chmod +x echo
export PATH=/tmp:$PATH
```
on passe de:
`
/usr/local/bin:/usr/bin:/bin
`
a: 
`/tmp:/usr/local/bin:/usr/bin:/bin`

et voila, prochaine fois quon execute `./level03`je vais obtenir les droits `flag03` dans un shell
`/bin/sh` et recuprere le flag `getflag`

# risk et fix
Un programme SUID ne doit jamais :
utiliser system()
dependre du $PATH
sinon → elevation de privileges garantie.
