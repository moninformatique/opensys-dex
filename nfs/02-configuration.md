# Configuration de NFS

La configuration NFS se fait dans un premier temps au niveau du serveur et dans un second temps au niveau du client.

## Configuration du serveur NFS

### Préparation du répertoire à partager

Pour commencer, nous devons préparer un répertoire de partage si cela n'est pas encore fait

- **1. Création du répertoire à partager**

Créez un répertoire qui sera le répertoire partager entre tous les clients.

```bash
mkdir /sharedfs
```

- **2. Attribution des permissions sur le répertoire**

Dans un premier temps, nous attribuons `nfsnobody` comme utilisateur et groupe propriétaire au répertoire précédemment crée. Puis, dans un second temps, nous accordons sur le répertoire les droits de lecture, d'écriture et d'exécution (rwx) à l'utilisateur et au groupe propriétaire et les droits de lecture et d'exécution (rx) aux  autres utilisateurs.

```bash
chown -R nfsnobody:nobody /sharedfs
chmod -R 775 /sharedfs
```

- **3. Configuration SELinux**

Nous configurons le contexte selinux NFS pour ce répertoire

```bash
semanage fcontext -a -t nfs_t "/sharedfs(/.*)?"
restorecon -Rv /sharedfs/
```

### Publication du répertoire de partage sur le réseau

Pour publier le répertoire de partage, nous ouvrons le fichier `/etc/exports` avec un éditeur de texte

> **NB :** Le fichier de configuration /etc/exports sert de liste de contrôle d'accès pour spécifier les systèmes de fichiers qui peuvent être exportés via NFS vers des clients autorisés. Il fournit des informations à mountd et au démon du serveur de fichiers NFS basé sur le noyau, nfsd.

```bash
vim /etc/exports
```

puis nous indiquons dans ce fichier les informations qu'il faut comme le répertoire à partagé, l'adresse IP du client et certains options à appliquer sur la connexion.

```text
/sharedfs    IP_client_NFS(rw,sync,no_root_squash)
```

- **/sharednfs** : Chemin absolu du répertoire à partager
- **IP_client_NFS** : L'adresse IP du/des clients NFS (séparés d'une virgule). (Utilisez (*) pour identifier toutes les adresses ou l'adresse réseau pour parler des machines d'un réseau spécifique)
- **rw** : Les permissions de lecture et d'écriture pour le client sur le répertoire
- **sync** : Pour synchroniser les modifications faites
- **no_root_squash** : Pour ne pas matcher les droits root à l'utilisateur qui accède au répertoire

Voici quelques options que vous pouvez utiliser :

| Options | Descriptions |
|---------|--------------|
| rw      | Permet la lecture et l'écriture sur le répertoire partagé |
| ro      | Permet la lecture uniquement sur le répertoire partagé (par défaut) |
| async   | Permet au serveur NFS de violer le protocole NFS et de répondre aux requêtes avant que les changements effectués par la requête aient été appliqués sur l'unité de stockage. Cette option améliore les performances mais a un coût au niveau de l'intégrité des données (données corrompues ou perdues) en cas de redémarrage non-propre (crash système).|
| sync    | Permet de répondre aux demandes uniquement après que les modifications ont été transférées dans le système de stockage. (par défaut) |
| root_squash | Force le mapping de l'utilisateur root (uid/gid = 0) vers l'utilisateur anonyme (option par défaut). L'option root_squash spécifie que le root de la machine cliente n'a pas les droits de root sur le répertoire partagé.|
| no_root_squash |  N'effectue pas de mapping pour l'utilisateur root. L'option no_root_squash spécifie que le root de la machine sur laquelle le répertoire est monté a les droits de root sur le répertoire.|
| all_squash    | Transformer tous les uid/gid en l'utilisateur anonyme. L'option inverse est no_all_squash, qui est celle par défaut.|
| no_all_squash | N'effectue pas de mapping de tous les uid/gid vers l'utilisateur anonyme|
|anonuid et anongid | Ces options définissent explicitement l'uid et le gid du compte anonyme. Cette option est principalement utile pour des clients PC/NFS, dans le cas où vous souhaiteriez que toutes les requêtes semblent provenir d'un seul et même utilisateur.|
|subtree_check | Si un sous-répertoire dans un système de fichiers est partagé, mais que le système de fichiers ne l'est pas, alors chaque fois qu'une requête NFS arrive, le serveur doit non seulement vérifier que le fichier accédé est dans le système de fichiers approprié (ce qui est facile), mais aussi qu'il est dans l'arborescence partagée (ce qui est plus compliqué). Cette vérification s'appelle subtree_check.|
|no_subtree_check | Cette option neutralise la vérification de sous-répertoires, ce qui a des subtiles implications au niveau de la sécurité, mais peut améliorer la fiabilité dans certains cas.|
|secure et insecure | Cette option impose l'utilisation d'un port réservé (inférieur à 1024) comme origine de la requête. Cette option est activée par défaut. Pour la désactiver, utilisez **insecure**. |
|no_wdelay | Cette option est sans effet si async est déjà active. Le serveur NFS va normalement retarder une requête d'écriture sur disque s'il suspecte qu'une autre requête en écriture liée à celle-ci soit en cours ou puisse survenir rapidement. Cela permet l'exécution de plusieurs requêtes d'écriture en une seule passe sur le disque, ce qui peut améliorer les performances. En revanche, si un serveur NFS reçoit principalement des petites requêtes indépendantes, ce comportement peut réellement diminuer les performances no_wdelay  permet de désactiver cette option. On peut explicitement spécifier ce comportement par défaut en utilisant l'option wdelay. |
|nohide | Cette option est basée sur l'option de même nom fournie dans le NFS d'IRIX. Normalement, si un serveur partage deux systèmes de fichiers dont un est monté sur l'autre, le client devra explicitement monter les deux systèmes de fichiers pour obtenir l'accès complet. S'il ne monte que le parent, il verra un répertoire vide à l'endroit où l'autre système de fichiers est monté. Ce système de fichiers est « caché ».|

On n'a plus qu'à appliquer les changements effectués

```bash
exportfs -ar
```

Quelques options utiles

|Options|Fonctions|
|-------|---------|
| -a | Export or unexport all directories |
| -r | Reexport all directories |
| -s |  Display the current export list suitable for /etc/exports|

### Déclaration des règles NFS sur le pare-feu

A présent, nous devons autoriser les connexions aux services nfs en ajoutant les règles suivanntes au pare-feu

```bash
firewall-cmd --permanent --zone=public --add-service={nfs,nfs3,mountd,rpc-bind}
firewall-cmd --reload
```

C'est ici que s’achève la configuration de votre serveur NFS

## Configuration du client NFS

### Préparation d'un point de montage

Nous créons un répertoire qui seras utilisé comme point de montage pour le répertoire partagé NFS.

```bash
mkdir -p /mnt/sharedfs
```

### Montage du répertoire distant

Nous interrogeons le serveur NFS afin d'obtenir les répertoires partagés que nous pouvons joindre depuis notre client

```bash
showmount -e IP_serveur_NFS
```

Si tout est bien configuré, nous devons avoir comme résultat le répertoire partagé du serveur NFS.

Quelques options utiles

|Options|Fonctions|
|-------|---------|
| -a | List  both  the  client  hostname or IP address and mounted directory in host:dir format. |
| -e | Show the NFS server's export list |

Nous n'avons plus qu'à monter le répertoire distant dans le système de fichiers du client

```bash
mount -t nfs IP_serveur_NFS:/sharedfs /mnt/sharedfs
```

Vous pouvez vérifier que le montage à été correctement effectué grâce à la commande

```bash
df -hT
```

### Test des installations

Pour tester les différentes permissions, nous pouvons créer un fichier dans le répertoire monté

```bash
touch /mnt/sharedfs/test_nfs.txt
```

Si cela fonctionne, nous venons de configurer correctement notre serveur ainsi que le client NFS

### Configuration d'un montage NFS permanent

Le premier montage du système de fichier distant s'est fait manuellement. Afin de le rendre automatique au démarrage de la machine cliente, il nous faut configurer le fichier `/etc/fstab`.

```bash
vim /etc/fstab
```

Nous ajoutons l'entrée suivante à la suite de ceux existant

```text
IP_serveur_NFS:/sharedfs   /mnt/sharedfs   nfs auto,_netdev,nofail,soft,retrans=2,timeo=5 0 0
```
