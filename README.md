# 🚀 Guide de Déploiement Automatique avec GitHub Actions

Ce dépôt contient un site web statique déployé automatiquement sur un serveur via GitHub Actions. Ce README détaille la configuration nécessaire pour mettre en place et utiliser ce système de déploiement continu.

## 📋 Présentation du Projet

Ce projet utilise GitHub Actions pour automatiser le déploiement d'un site web HTML/CSS sur un serveur distant via SSH et rsync. Chaque fois qu'un push est effectué sur la branche `main`, le workflow se déclenche et met à jour le site web sur le serveur.

## ⚙️ Configuration des Secrets GitHub

Pour que le workflow fonctionne correctement, vous devez configurer les secrets suivants dans votre dépôt GitHub :

1. Allez dans votre dépôt GitHub → Settings → Secrets and variables → Actions
2. Cliquez sur "New repository secret" et ajoutez les secrets suivants :

| Nom du Secret | Description | Exemple |
|--------------|-------------|---------|
| `SSH_PRIVATE_KEY` | Contenu de votre clé privée SSH | Contenu du fichier `~/.ssh/id_rsa` |
| `SSH_USER` | Nom d'utilisateur pour votre serveur | `ubuntu`, `root`, etc. |
| `SSH_HOST` | Adresse IP ou nom de domaine du serveur | `12.34.56.78` ou `monserveur.com` |
| `REMOTE_DIR` | Chemin du répertoire de déploiement | `/var/www/html/monsite` |

> 💡 **Note**: Contrairement à de nombreux tutoriels, vous n'avez pas besoin de configurer manuellement `SSH_KNOWN_HOSTS`. Notre workflow récupère automatiquement l'empreinte du serveur pendant l'exécution.

> 🔑 **Astuce pour les clés SSH**: Si vous vous connectez déjà à votre serveur avec une clé SSH existante, vous pouvez réutiliser cette même clé pour GitHub Actions! Il vous suffit de copier le contenu de votre fichier de clé privée (généralement `~/.ssh/id_rsa` ou `~/.ssh/id_ed25519`) dans le secret `SSH_PRIVATE_KEY`. Aucune configuration supplémentaire n'est nécessaire sur le serveur car votre clé est déjà autorisée.

## 🔧 Prérequis sur votre Serveur de Déploiement

**Important** : Ces étapes doivent être réalisées sur votre serveur de déploiement **avant** d'exécuter le workflow GitHub Actions. Elles sont nécessaires pour que le serveur autorise la connexion depuis le runner GitHub.

### Option 1 : Utiliser une clé SSH existante (recommandé si vous avez déjà accès SSH)

Si vous vous connectez déjà à votre serveur avec une clé SSH :

```bash
# Aucune configuration supplémentaire nécessaire sur le serveur
# Utilisez simplement le contenu de votre clé privée existante (~/.ssh/id_rsa) comme valeur pour SSH_PRIVATE_KEY dans GitHub
```

### Option 2 : Créer une nouvelle paire de clés dédiée à GitHub Actions

Si vous préférez créer une nouvelle paire de clés dédiée à GitHub Actions :

```bash
# Sur votre machine locale : Générer une nouvelle paire de clés SSH
ssh-keygen -t ed25519 -C "github-actions-deploy"

# Sur votre serveur : Ajouter la clé publique aux clés autorisées
echo "votre-clé-publique-ssh" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Configuration du serveur (nécessaire dans tous les cas)

```bash
# 1. Installer rsync si ce n'est pas déjà fait
sudo apt-get update && sudo apt-get install -y rsync

# 2. Créer le répertoire de destination
sudo mkdir -p /var/www/html/monsite

# 3. Attribuer les permissions nécessaires
sudo chown -R votre-utilisateur:www-data /var/www/html/monsite
sudo chmod -R 755 /var/www/html/monsite
```

> ⚠️ **Attention** : Ces commandes sont à exécuter sur votre serveur de déploiement, pas sur les runners GitHub. Elles configurent votre serveur pour qu'il accepte les déploiements provenant de GitHub Actions.

## 🔄 Structure du Workflow

Le workflow GitHub Actions (`.github/workflows/deploy.yml`) contient les étapes suivantes, exécutées automatiquement sur les runners GitHub :

1. **Déclenchement** : Activé lors d'un push sur la branche `main`
2. **Récupération du code source** : Récupère le code source du dépôt
3. **Configuration SSH** : Met en place l'authentification SSH sécurisée
4. **Récupération automatique de l'empreinte du serveur** : Obtient l'empreinte du serveur via `ssh-keyscan`
5. **Déploiement vers le serveur** : Transfère les fichiers via rsync avec les exclusions appropriées
6. **Statut du déploiement** : Affiche un message de succès une fois terminé

## 🔍 Test et Dépannage

### Tester le workflow

1. Effectuez un commit et un push sur la branche `main`
2. Allez dans l'onglet "Actions" de votre dépôt GitHub
3. Cliquez sur l'exécution la plus récente du workflow pour voir les détails
4. Vérifiez si toutes les étapes se sont exécutées avec succès

### Résolution des problèmes courants

| Problème | Solution |
|----------|----------|
| Erreur d'authentification SSH | Vérifiez que `SSH_PRIVATE_KEY` est correctement configuré et que la clé publique est présente dans `authorized_keys` sur votre serveur |
| Permission denied | Vérifiez les permissions du dossier de destination sur le serveur |
| Échec du déploiement rsync | Vérifiez que rsync est installé sur le serveur et que les chemins sont corrects |
| Problèmes avec l'empreinte du serveur | Si vous rencontrez des problèmes avec la vérification automatique du serveur, vérifiez que `SSH_HOST` est correct |

## 🔧 Personnalisation

### Adapter le workflow pour d'autres projets

Pour réutiliser ce workflow avec d'autres projets web :

1. Conservez la même structure de secrets GitHub
2. Modifiez les exclusions dans la commande rsync selon vos besoins
3. Adaptez les branches déclenchant le déploiement dans la section `on.push.branches`
4. Ajoutez des étapes supplémentaires si nécessaire (minification, tests, etc.)

### Exemples d'extensions possibles

- Intégrer des tests automatiques avant le déploiement
- Ajouter des notifications Slack/Discord après le déploiement
- Configurer un déploiement multi-environnement (staging/production)
- Implémenter une invalidation de cache CDN après déploiement

## 📚 Ressources Officielles

- [GitHub Actions - Documentation](https://docs.github.com/en/actions)
- [Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [webfactory/ssh-agent Action](https://github.com/webfactory/ssh-agent)
- [actions/checkout Action](https://github.com/actions/checkout)
- [Rsync Documentation](https://linux.die.net/man/1/rsync)
- [Guide SSH pour GitHub Actions](https://docs.github.com/en/actions/guides/using-ssh-keys-in-a-workflow)
- [SSH-Keyscan Manual](https://man.openbsd.org/ssh-keyscan.1)
- [GitHub Actions - Variables d'environnement](https://docs.github.com/en/actions/learn-github-actions/environment-variables)

---

💡 **Astuce finale** : Si vous modifiez ce workflow, testez-le d'abord avec un répertoire de destination non critique sur votre serveur avant de cibler votre environnement de production.