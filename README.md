# caddy-cloudflare

Ce dépôt compile Caddy avec le module DNS Cloudflare (`github.com/caddy-dns/cloudflare`).
Le binaire sert à obtenir des certificats par défi DNS ACME chez Cloudflare.
Chaque release contient le binaire, ses empreintes et une signature minisign.

## Fichiers d'une release

- `caddy-linux-amd64` : binaire statique pour linux/amd64.
- `SHA256SUMS` : empreinte SHA-256 du binaire.
- `SHA256SUMS.minisig` : signature minisign de `SHA256SUMS`.

## Politique de version

- Seule la dernière version stable de Caddy est compilée. Pas de bêta, pas de rc.
- Seule la majeure 2 est acceptée.
- Une version est compilée 7 jours après sa publication par Caddy.
- Le workflow tourne chaque lundi à 03:17 UTC.
- Une version déjà publiée ici n'est pas recompilée.
- Le module Cloudflare est pris à sa dernière version taguée.

## Compiler à la main

Une version exacte, sans délai de 7 jours :

```
gh workflow run build.yml --repo iBroStudio/caddy-cloudflare -f version=2.11.7
```

Sans version, le workflow applique la politique ci-dessus :

```
gh workflow run build.yml --repo iBroStudio/caddy-cloudflare
```

## Vérifier une release

Télécharger les trois fichiers de la release dans un même dossier.
Avoir `minisign.pub` de ce dépôt à portée.
Vérifier la signature, puis l'empreinte :

```
minisign -Vm SHA256SUMS -p minisign.pub
sha256sum -c SHA256SUMS
```

Sous macOS, utiliser `shasum -a 256 -c SHA256SUMS`.

## Rotation de clé

La clé privée vit uniquement dans le secret GitHub `MINISIGN_SECRET_KEY`.
Elle n'existe nulle part ailleurs.
Si elle est perdue ou compromise, on régénère une paire.

1. Générer une paire sans mot de passe, la clé privée hors du dépôt :
   `minisign -G -W -p minisign.pub -s <dossier temporaire>/minisign.key`.
2. Remplacer le secret :
   `gh secret set MINISIGN_SECRET_KEY --repo iBroStudio/caddy-cloudflare < <dossier temporaire>/minisign.key`.
3. Supprimer la clé privée locale.
4. Commiter le nouveau `minisign.pub`.
5. Redéployer la clé publique sur les serveurs qui vérifient les releases.

Les releases déjà publiées restent signées par l'ancienne clé.
