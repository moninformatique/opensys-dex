# Installation de NFS

Dans cette documentation, nous aurons à déployer un serveur NFS sur un système RockyLinux 8.

>**NB** : L'ensemble des commandes utilisées devrons être lancer en tant que root.

## Installation d'un serveur NFS

### Installation des paquets

Pour commençer, nous mettons à jours les paquets avant d'installer les outils pour NFS sur le serveur

```bash
dnf update -y
dnf install nfs-utils -y
```

### Activation des services

Après l'installtion, nous activons les différents services de nfs pourqu'ils puissent être automatiquement mis en marche lors du démarage/redémarrage du système. L'option `--now` permettra de démarrer le service immédiatement.

```bash
systemctl enable --now rpcbind
systemctl enable --now nfs-server
```

## Installation d'un client NFS

L'installation d'un client NFS cse fait en installant uniquement les outils NFS.

```bash
dnf update -y
dnf install nfs-utils -y
```
