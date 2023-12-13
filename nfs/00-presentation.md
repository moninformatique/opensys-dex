# Présentation de NFS

Le NFS, pour Network File System, est un protocole permettant à un ordinateur d'accéder à des fichiers extérieurs via un réseau. Développé dans les années 80, le NFS s'est ensuite imposé comme la norme en matière de serveur de fichiers. Focus sur le NFS et notamment sa dernière version, le NFSv4.

## Qu'est-ce que le Network File System ?

Le NFS permet à un utilisateur d’accéder, via son ordinateur (le client), à des fichiers stockés sur un serveur distant. Il est possible de consulter mais aussi mettre à jour ces fichiers, comme s’ils étaient présents sur l’ordinateur client (c’est-à-dire comme des fichiers locaux classiques). Des ressources peuvent ainsi être stockées sur un serveur et accessibles via un réseau par une multitude d’ordinateurs connectés. Ce système a été développé par l’ancien constructeur d’ordinateurs Sun Microsystems en 1984, aujourd’hui fusionné avec Oracle. Le NFS a été créé pour les systèmes Unix. Il existe maintenant des versions pour Windows et Mac OS.

Comme pour la plupart des concepts de réseau, NFS comporte une partie client et une partie serveur. Le côté serveur est constitué du système qui exporte (partage) des systèmes de fichiers vers d'autres systèmes. Le côté client est constitué des systèmes qui ont besoin d'accéder au système de fichiers exporté par le serveur.
