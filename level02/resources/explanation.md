# Explications

Le level02 nous donne un fichier ```level02.pcap``` , c'est un fichier contenant une capture du trafic reseau (les echanges entre une machine et un serveur).<br>

Ce fichier s'ouvre avec des outils prevu pour , comme Wireshark

Apres s'etre donne les droits pour ouvrir le fichier, on l'ouvre avec Wireshark et on a une liste des echanges de paquets avec pleins d'infos comme la source, la destination, le protocole et le contenu de ces paquets.

Pour pouvoir exploiter ceci et rendre lisible le contenu on va faire un clique droit sur un des paquets et `Follow -> TCP Stream`, cela nous montre une version lisible en ASCII du contenu.

On y voit ce que le client a echange (en rouge) avec le serveur (en bleu), ce qui nous interesse se trouve ici :
```sh
Password:
ft_wandr...NDRel.L0L
```

Mais en essayant ce mot de passe pour le level02 cela ne fonctionne pas..

Il faut aller plus loin, on peut trouver les "." etranges dans le mot de passe et en effet ils le sont. Les points ici ne sont peut etre pas de vrais points mais d'autres caracteres non affichables que Wireshark remplace par des points.

Pour verifier la vrai signification de ces points il nout faut passer en Hex Dump dans Wireshark et observer la valeur hexadecimale et chaque input :
```sh
000000B9  66                                                 f
000000BA  74                                                 t
000000BB  5f                                                 _
000000BC  77                                                 w
000000BD  61                                                 a
000000BE  6e                                                 n
000000BF  64                                                 d
000000C0  72                                                 r
000000C1  7f                                                 .
000000C2  7f                                                 .
000000C3  7f                                                 .
000000C4  4e                                                 N
000000C5  44                                                 D
000000C6  52                                                 R
000000C7  65                                                 e
000000C8  6c                                                 l
000000C9  7f                                                 .
000000CA  4c                                                 L
000000CB  30                                                 0
000000CC  4c                                                 L
000000CD  0d                                                 .
```

En regardant une table ASCII on remarque et un `.` vaut `2e`, que `7f` correspond a `DEL` et `0d` au caractere `NULL` ce qui signifie que on peut reconstituer ce que a fait le client :

- ft_wandr<br>
- 3x DEL -> ft_wa<br>
- ft_waNDRel
- 1x DEL -> ft_waNDRe
- ft_waNDReL0L

Le mot de passe est donc `ft_waNDReL0L`
