### Explication

En regardant le fichier dans /etc/passwd on remarque une sorte de mot passe chiffre derriere l' user flag01

On extrait le fichier de la VM grace a scp , ```scp -P 4242 level01@192.168.56.103:/etc/passwd .```

On execute John The Ripper dessus ```john passwd``` et ensuite on va regarder le resultat dans ```john.pot```

42hDRfypTqqnw:abcdefg

