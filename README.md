# bastiengrandjean.dev

Portfolio de Bastien Grandjean, développeur Ruby on Rails. Site 100 % statique :
une page par langue (`/` en français, `/en/` en anglais), aucun JavaScript,
aucun framework CSS.

## Stack

HTML et CSS écrits à la main, servis par nginx dans un conteneur Docker,
déployés avec [Kamal](https://kamal-deploy.org) sur le même serveur Hetzner
que [movethespark.com](https://movethespark.com) et
[cookitgreen.com](https://cookitgreen.com). Polices (Newsreader, Geist, Geist
Mono) auto-hébergées en WOFF2, images en WebP avec repli JPEG.

## Lancer en local

```bash
docker build -t portfolio .
docker run --rm -p 8080:80 portfolio
```

Ouvrir `http://localhost:8080`. La page anglaise est sur `/en/`, la 404 sur
n'importe quelle URL inconnue.

Pour itérer sans reconstruire l'image à chaque changement, servir le dossier
`site/` directement (`cd site && python3 -m http.server 8080`) — les chemins
sont tous relatifs, ça fonctionne pareil.

## Déployer

Nécessite `KAMAL_REGISTRY_PASSWORD` (jeton GitHub, scope `write:packages`) et
`PORTFOLIO_SERVER_IP` dans l'environnement du shell qui lance Kamal (jamais
commités, voir `.kamal/secrets`) :

```bash
export KAMAL_REGISTRY_PASSWORD=$(gh auth token)
export PORTFOLIO_SERVER_IP=<IP du serveur>
bin/kamal setup    # première fois seulement
bin/kamal deploy   # ensuite
```

Avant le premier déploiement, pointer les enregistrements DNS **A** de
`bastiengrandjean.dev` et `www.bastiengrandjean.dev` vers cette IP.
`www` redirige vers l'apex en 301 (géré par nginx, voir `nginx.conf`).

## Ajouter un projet

1. Dupliquer un `<article class="project">` dans `site/index.html` **et**
   `site/en/index.html` (les deux pages ne partagent pas de template, il faut
   répercuter le changement des deux côtés).
2. Déposer les captures dans `site/assets/img/` et générer leur version
   WebP : `cwebp -q 82 fichier.jpg -o fichier.webp`.
3. Mettre à jour `site/sitemap.xml` si une nouvelle page est créée.

## Notes

- `site/cv.pdf` et `site/en/cv.pdf` ne sont pas encore fournis — voir le
  dernier message de la session qui a construit ce dépôt pour le détail des
  points encore en attente côté contenu. La CI (`check-links`) échouera tant
  que ces deux fichiers n'existent pas : c'est volontaire, pas un bug.
- La photo source (432×540) est plus petite qu'idéal pour un hero aussi
  grand ; une version en plus haute résolution améliorerait le rendu.
- `.htmlvalidate.json` désactive la règle `no-redundant-role` : les listes à
  puces supprimées (`.gallery`, `.stack`, `.contact__links`) portent un
  `role="list"` explicite pour contourner un bug connu de VoiceOver/Safari qui
  leur retire sinon leur sémantique de liste — un validateur HTML pur ne voit
  pas le CSS et le considère (à raison, hors de ce contexte) comme redondant.
