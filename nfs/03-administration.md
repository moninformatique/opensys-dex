# Administration

## Activation/Désactivation du mode debug

rpcdebug est un outil en ligne de commande qui permet d'activer et désactiver le débogage de plusieurs modules nfs, ainsi que diverses catégories de journalisation de débogage au sein de ces modules.  L'option "-m" identifie le module pour lequel le débogage est activé, et l'option "-s" identifie les flags de débogage qui seront activés.

- Débogage pour un serveur NFS

```bash
rpcdebug -m nfsd -s all
```

- Débogage pour un client NFS

```bash
rpcdebug -m nfs -s all
```

- Débogage pour le protocole RPC

Dans de nombreux cas, le protocole RPC doit également être débogué, ce qui peut être réalisé en utilisant : `rpcdebug -m rpc -s all`

>**NB :** Une fois le débogage activé, vous trouverez les logs dans le fichier `/var/log/messages` ou grâce à la commande `dmesg`.

- Désactivation du débogage

Pour désactiver le débogage, il faut utiliser l'option `-c` qui effacera le flag activé en question.

```bash
rpcdebug -m nfsd -c all
rpcdebug -m nfs -c all
rpcdebug -m rpc -c all
```
