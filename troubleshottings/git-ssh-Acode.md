# Git + SSH dans Acode (Android)

## Contexte

Objectif : utiliser GitHub depuis le terminal Acode avec SSH,
comme sur un PC Linux.

Environnement :
- Acode
- Alpine Linux
- OpenSSH
- Git
- GitHub

---

## 1. Problème rencontré

Acode utilisait Dropbear comme client SSH alors que
`ssh-keygen` provenait d'OpenSSH.

Vérification :

    which ssh
    /usr/bin/ssh

    ls -l /usr/bin/ssh
    /usr/bin/ssh -> dbclient

Paquets présents :

    dropbear-dbclient
    dropbear-ssh
    openssh-keygen

Conséquence :
`ssh` = Dropbear
`ssh-keygen` = OpenSSH

---

## 2. Solution

Supprimer Dropbear :

    apk del dropbear-dbclient dropbear-ssh

Installer le client OpenSSH :

    apk add openssh-client-default

Vérifier :

    ssh -V

Doit afficher :

    OpenSSH_...

---

## 3. Réinitialisation des clés

Les clés SSH utilisateur sont stockées dans :

    ~/.ssh/

Pour repartir complètement de zéro dans cet environnement :

    rm -rf ~/.ssh

Puis générer une nouvelle paire :

    ssh-keygen -t ed25519

Fichiers créés :

    ~/.ssh/id_ed25519       # clé privée
    ~/.ssh/id_ed25519.pub   # clé publique

Ne jamais partager `id_ed25519`.

---

## 4. GitHub

Ajouter uniquement :

    ~/.ssh/id_ed25519.pub

dans :

GitHub → Settings → SSH and GPG keys

Tester :

    ssh -T git@github.com

---

## 5. Repository

Utiliser l'URL SSH :

    git@github.com:USER/REPOSITORY.git

et non l'URL HTTPS si l'objectif est d'utiliser SSH.

Exemple :

    git clone git@github.com:USER/REPOSITORY.git

Vérifier le remote :

    git remote -v

---

## 6. Particularité Acode

Le terminal Acode est un environnement Linux isolé.

Le `root` d'Acode n'est pas le root Android.

L'accès direct à :

    /storage/emulated/0/

peut donc être refusé.

Solution retenue :
stocker les projets dans l'environnement Acode :

    ~/projects/

Acode peut accéder à ce dossier.

---

## État final

Acode
  ↓
Alpine Linux
  ↓
OpenSSH
  ↓
Git
  ↓
GitHub

Remote Git :

    git@github.com:USER/REPOSITORY.git
    
    ## Vérification finale

    ssh -T git@github.com
    git remote -v
    git status

Résultat attendu :
- SSH authentifié
- remote en SSH
- repository fonctionnel