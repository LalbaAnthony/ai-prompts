# Mission : audit Open Graph d'un WordPress standard construit avec Elementor

Tu travailles sur un projet WordPress classique (arborescence standard `wp-admin/`, `wp-includes/`, `wp-content/`, sans Bedrock ni Composer), dont les pages sont construites avec Elementor (et éventuellement Elementor Pro). Objectif : auditer la présence et la qualité des balises Open Graph (et Twitter Cards associées), attribuer un score, et produire un plan d'actions si le résultat est insuffisant.

Le MCP Playwright (Chromium) est déjà configuré et opérationnel. Utilise-le pour le rendu navigateur. Utilise `curl` pour le HTML brut.

## Contraintes

- Audit en **lecture seule**. Aucune modification de code, de base de données, de plugin, de template Elementor ou de configuration sans validation explicite.
- Ne jamais toucher au core : `wp-admin/`, `wp-includes/`, fichiers `wp-*.php` à la racine.
- Ne jamais modifier directement un plugin, un thème parent ou Elementor : ces fichiers sont écrasés à chaque mise à jour.
- Ne jamais ouvrir ni enregistrer une page dans l'éditeur Elementor (un simple enregistrement réécrit `_elementor_data` et régénère le CSS).
- Toute commande WP-CLI doit être non destructive (`get`, `list`, `option get`, `post meta get`, `eval` en lecture, `db query` limité à `SELECT`). Interdits : `elementor flush-css`, `elementor replace-urls`, `cache flush`, `rewrite flush`, `search-replace`, `transient delete`.
- Si l'environnement local ne répond pas, t'arrêter et le signaler. Ne pas auditer une URL de production sans instruction explicite.

## Phase 1 — Reconnaissance du projet

1. Localiser la racine WordPress (dossier contenant `wp-config.php`, parfois un niveau au-dessus de la racine web). Vérifier si le projet est versionné (`git status`) et ce qui est suivi.
2. Lire `wp-config.php` **sans jamais afficher les secrets** (identifiants de base, clés et salts, clés de licence) : récupérer `WP_HOME`, `WP_SITEURL`, `WP_ENVIRONMENT_TYPE`, `WP_DEBUG`, `WP_CACHE`, `$table_prefix`, et toute constante personnalisée. Si `WP_HOME` n'est pas défini, utiliser `wp option get home`. Cette valeur est l'URL de base de l'audit.
3. Relever toute logique conditionnelle d'environnement dans `wp-config.php` ou dans un fichier inclus (`wp-config-local.php`, etc.).
4. Inventorier sans WP-CLI si nécessaire : `wp-content/plugins/` (lire l'en-tête `Plugin Name` / `Version` de chaque plugin), `wp-content/themes/` (thème parent et thème enfant via `style.css`, champ `Template`), `wp-content/mu-plugins/`, drop-ins (`wp-content/advanced-cache.php`, `object-cache.php`, `db.php`).
5. Identifier les sources potentielles de balises OG :
   - Plugins SEO : Yoast, Rank Math, SEOPress, All in One SEO, The SEO Framework, Slim SEO, Jetpack (module Publicize/OG), plugins de partage social.
   - Thème parent et enfant (`wp-content/themes/*`) : chercher `og:`, `property="og`, `twitter:`, `wp_head`, `wpseo_`, `rank_math/`, `seopress_`, `name="description"`. Si le thème est **Hello Elementor**, vérifier son réglage d'émission de la meta description (page de réglages du thème) : source fréquente de doublon avec le plugin SEO.
   - Mu-plugins (`wp-content/mu-plugins/`) et `functions.php` du thème enfant.
   - **Elementor Pro — Code personnalisé** : `wp post list --post_type=elementor_snippet --fields=ID,post_title,post_status`, puis inspecter le code et sa position (`<head>`, début/fin de `<body>`).
   - **Contenu Elementor** : rechercher des balises injectées via widgets HTML ou shortcodes dans les templates et pages :
     ```bash
     wp db query "SELECT post_id FROM $(wp db prefix)postmeta WHERE meta_key='_elementor_data' AND (meta_value LIKE '%og:%' OR meta_value LIKE '%twitter:%')"
     ```
     Une meta placée dans le `<body>` par un widget est hors `<head>` : la signaler comme source parasite.
   - Addons Elementor (Essential Addons, Crocoblock/JetEngine, Happy Addons, etc.) et leurs éventuels modules SEO ou de partage.
6. Si WP-CLI est disponible (`wp --info`, éventuellement `wp --path=<racine>`) : `wp plugin list --status=active`, `wp plugin list --status=must-use`, `wp theme list --status=active`, `wp post-type list --public=1`, `wp taxonomy list --public=1`, et récupérer les options OG du plugin SEO actif (image par défaut, activation OG, activation Twitter Cards, prise en compte du contenu Elementor).
7. Elementor spécifique :
   - Versions : `wp plugin get elementor --field=version`, idem `elementor-pro`.
   - **Mode maintenance / bientôt disponible** : `wp option get elementor_maintenance_mode_mode` (`maintenance` renvoie un 503 aux visiteurs non connectés, `coming_soon` affiche un template). Tout mode actif = crawlers sociaux bloqués → défaut critique.
   - **Theme Builder (Pro)** : lister les templates `wp post list --post_type=elementor_library --fields=ID,post_title,post_status` et leurs conditions d'affichage (`wp post meta get <ID> _elementor_conditions`, `wp post meta get <ID> _elementor_template_type`). Identifier quels templates `header`, `single`, `single-post`, `single-page`, `archive`, `product`, `search-results`, `error-404` remplacent les templates du thème.
   - Pages utilisant le template `elementor_canvas` ou `elementor_header_footer` (`_wp_page_template`) : vérifier que `wp_head` reste exécuté.
   - Vérifier que `elementor_library` et les post types internes Elementor ne sont ni publics ni présents dans le sitemap.
8. Détecter les plugins de cache (WP Rocket, LiteSpeed Cache, W3 Total Cache, WP Super Cache, cache hébergeur), de minification/optimisation (Autoptimize, Perfmatters, retard d'exécution JS), de consentement cookies, de sécurité/WAF (Wordfence, iThemes/Solid Security, Cloudflare) susceptibles d'altérer le `<head>`, de différer le rendu ou de bloquer les crawlers sociaux.

Livrable intermédiaire : tableau des sources OG détectées (plugin, thème, mu-plugin, snippet Elementor, widget Elementor, addon), avec risque de doublon si plusieurs sources émettent des balises.

## Phase 2 — Échantillonnage des URLs

Construire un échantillon représentatif, via le sitemap (`/wp-sitemap.xml`, `/sitemap_index.xml`, `/sitemap.xml`) ou WP-CLI :

- Page d'accueil (et page des articles si distincte)
- 3 articles minimum (dont le plus récent, un sans image mise en avant si possible, un avec titre long)
- 3 pages minimum, dont au moins **une construite avec Elementor** (`_elementor_edit_mode` = `builder`) et, si elle existe, **une non construite avec Elementor**
- 1 page sur template `elementor_canvas` si présente, 1 landing page Elementor (`e-landing-page`) si présente
- 1 URL par template Theme Builder `single` / `archive` actif, si Elementor Pro est présent
- 2 éléments par custom post type public (y compris ceux créés par JetEngine, ACF, CPT UI)
- 1 archive de catégorie, 1 archive d'étiquette, 1 archive par taxonomie personnalisée publique
- 1 archive auteur, 1 archive de date
- Page de recherche (`/?s=test`), page 404
- Pages WooCommerce si présent (boutique, produit, catégorie produit)
- Variantes de langue si plugin multilingue (WPML, Polylang, TranslatePress)

Pour lister les pages Elementor :

```bash
wp post list --post_type=page,post --meta_key=_elementor_edit_mode --meta_value=builder --fields=ID,post_title,url --format=csv
```

Plafond : 40 URLs. Consigner la liste finale avec, pour chaque URL, l'indication Elementor / non Elementor et le template Theme Builder appliqué.

## Phase 3 — Collecte

Pour chaque URL, deux collectes distinctes.

### A. HTML brut (ce que voient réellement les crawlers sociaux, qui n'exécutent pas le JavaScript)

```bash
curl -sL -A "facebookexternalhit/1.1 (+http://www.facebook.com/externalhit_uatext.php)" -o page.html -w "%{http_code} %{url_effective}\n" "<URL>"
curl -sL -A "Twitterbot/1.0" -o page_tw.html -w "%{http_code}\n" "<URL>"
curl -sL -A "LinkedInBot/1.0" -o page_li.html -w "%{http_code}\n" "<URL>"
```

Parser les `<meta>` du `<head>` **et** du `<body>` séparément (script Python ou Node dans un dossier temporaire hors du projet et hors de la racine web). Relever le code HTTP, les redirections, et toute page de challenge WAF, mur de consentement ou page de maintenance Elementor.

### B. DOM rendu via Playwright MCP

`browser_navigate` vers l'URL, puis `browser_evaluate` avec :

```js
() => {
  const metas = [...document.head.querySelectorAll('meta[property], meta[name]')]
    .map(m => ({
      key: m.getAttribute('property') || m.getAttribute('name'),
      content: m.getAttribute('content')
    }))
    .filter(m => /^(og:|article:|profile:|product:|twitter:|fb:)/.test(m.key)
      || ['description', 'robots'].includes(m.key));
  // Meta tags outside <head> (e.g. injected by an Elementor HTML widget)
  const bodyMetas = [...document.body.querySelectorAll('meta[property], meta[name]')]
    .map(m => m.getAttribute('property') || m.getAttribute('name'))
    .filter(k => /^(og:|twitter:|description)/.test(k));
  return {
    url: location.href,
    title: document.title,
    lang: document.documentElement.lang,
    canonical: document.querySelector('link[rel="canonical"]')?.href ?? null,
    elementor: {
      pageId: document.body.className.match(/elementor-page-(\d+)/)?.[1] ?? null,
      kit: document.body.className.match(/elementor-kit-(\d+)/)?.[1] ?? null,
      themeBuilderTemplates: [...document.querySelectorAll('[data-elementor-type]')]
        .map(el => ({ type: el.dataset.elementorType, id: el.dataset.elementorId }))
    },
    metas,
    bodyMetas,
    duplicates: Object.entries(
      metas.reduce((acc, m) => ((acc[m.key] = (acc[m.key] || 0) + 1), acc), {})
    ).filter(([key, count]) => count > 1 && !['og:image', 'og:locale:alternate', 'article:tag'].includes(key))
  };
}
```

Comparer A et B : toute balise présente uniquement dans le DOM rendu est **invisible pour les réseaux sociaux** → défaut critique.

### C. Vérification des images

Pour chaque `og:image` et `twitter:image` distincte :

```bash
curl -sI -A "facebookexternalhit/1.1" "<IMAGE_URL>"
```

Puis télécharger et mesurer les dimensions réelles (Python + Pillow, ou `identify`). Relever : code HTTP, `Content-Type`, poids, largeur, hauteur, ratio.

Point d'attention Elementor : les pages construites avec Elementor affichent souvent leurs visuels via des widgets ou des arrière-plans de section, sans image mise en avant. Vérifier pour chaque page Elementor si une image mise en avant existe (`wp post meta get <ID> _thumbnail_id`) et si l'`og:image` émise est spécifique ou retombe sur l'image par défaut.

## Phase 4 — Grille d'évaluation

### Défauts critiques (un seul suffit à déclencher le plan d'actions)

- `og:title`, `og:type`, `og:url` ou `og:image` absent sur un type de page indexable
- Balises OG présentes uniquement après exécution JS
- Crawler social bloqué (403, 503, challenge, mur de consentement, mode maintenance ou « bientôt disponible » Elementor dans le HTML brut)
- `og:image` inaccessible (≠ 200), URL relative, ou `Content-Type` non image
- Balises dupliquées avec valeurs divergentes (plusieurs sources concurrentes, y compris snippet ou widget Elementor)
- `og:url` pointant vers un autre domaine, un environnement de staging, ou en `http://` sur un site HTTPS (fréquent après migration sans remplacement d'URL dans les données Elementor)

### Critères de qualité par URL (score /100)

| Critère                       | Poids | Règle                                                                                                                                                                     |
| ----------------------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| og:title                      | 15    | Présent, non vide, spécifique à la page, 40–90 caractères, sans entité HTML non décodée (`&#8217;`, `&amp;amp;`)                                                          |
| og:description                | 15    | Présente, 100–200 caractères, spécifique à la page, pas de shortcode, pas de résidu Elementor (texte de bouton, libellé de widget, CSS), pas de texte tronqué brutalement |
| og:image                      | 20    | Absolue, HTTPS, 200, ≥ 1200×630, ratio ≈ 1.91:1 (tolérance ±10 %), < 5 Mo, JPG/PNG/WebP                                                                                   |
| og:image:width / height / alt | 5     | Présents et cohérents avec l'image réelle                                                                                                                                 |
| og:url                        | 10    | Absolue, identique à la canonical, sans paramètres de tracking                                                                                                            |
| og:type                       | 5     | Cohérent : `website` (accueil, archives), `article` (articles), `product` (produits), `profile` (auteurs)                                                                 |
| og:site_name                  | 5     | Présent, identique sur tout le site                                                                                                                                       |
| og:locale                     | 5     | Présent, format `fr_FR`, cohérent avec `<html lang>` ; `og:locale:alternate` si multilingue                                                                               |
| article:*                     | 5     | Sur les articles : `published_time`, `modified_time` au format ISO 8601 ; `author`, `section` si pertinents                                                               |
| Twitter Cards                 | 10    | `twitter:card` = `summary_large_image` si image grand format ; `twitter:title`/`description`/`image` présents ou héritage OG valide                                       |
| Unicité                       | 5     | Titre, description et image non dupliqués à l'identique sur des pages de contenu différentes                                                                              |

Pages exclues de la pénalité d'absence : 404, recherche, pages en `noindex` (le signaler néanmoins).

### Score global

- Moyenne pondérée des scores par URL, moyenne par type de page, et moyenne comparée **pages Elementor / pages non Elementor**.
- Seuils : **≥ 85** bon, **70–84** correct avec corrections ciblées, **< 70** insuffisant.
- Plan d'actions obligatoire si score global < 70 **ou** au moins un défaut critique **ou** un type de page complet < 60.

## Phase 5 — Rapport

Créer le dossier `audit/open-graph/` **hors de la racine web publique** (dans une installation WordPress standard, la racine du projet est généralement servie par le serveur web : un rapport placé à côté de `wp-config.php` serait accessible publiquement). Si aucun emplacement hors racine web n'est disponible, t'arrêter et le signaler. Si le projet est versionné, vérifier que ce dossier est ignoré par Git, sinon le signaler. Contenu :

1. `report.md` en français :
   - Synthèse : score global, verdict, 3 à 5 constats majeurs
   - Sources OG détectées et conflits
   - Scores par type de page et comparaison Elementor / non Elementor
   - Cartographie des templates Theme Builder appliqués
   - Tableau détaillé par URL : score, défauts, valeurs relevées
   - Écarts HTML brut / DOM rendu
   - Analyse des images
   - Plan d'actions (si déclenché)
2. `data.json` : données brutes de toutes les collectes.
3. `screenshots/` : uniquement pour les cas anormaux (mur de consentement, page de challenge, page de maintenance).

## Phase 6 — Plan d'actions (si déclenché)

Structurer par priorité :

- **P0 — Bloquant** : défauts critiques. Corriger avant tout partage social.
- **P1 — Majeur** : types de page < 70, images non conformes, absence de Twitter Cards.
- **P2 — Optimisation** : longueurs, unicité, métadonnées secondaires.

Pour chaque action : problème constaté, URLs ou types concernés, cause identifiée (fichier, plugin, option, template ou snippet Elementor), correction proposée, effort estimé (S/M/L), méthode de vérification.

Respecter les conventions d'un WordPress standard avec Elementor dans les corrections proposées :

- Ajout de plugin : depuis le répertoire officiel via l'administration ou `wp plugin install <slug> --activate` ; jamais de plugin nulled ni d'upload d'archive d'origine non vérifiée. Si le projet est versionné, préciser l'impact sur le dépôt.
- Code custom transverse : mu-plugin dans `wp-content/mu-plugins/`. Code lié au thème : `functions.php` du **thème enfant** (le créer si seul le thème parent est actif). Jamais dans le thème parent, Elementor ou un plugin tiers.
- Si un plugin SEO est actif : corriger via sa configuration ou ses filtres (`wpseo_opengraph_*`, `rank_math/opengraph/*`, `seopress_social_*`, etc.) plutôt que d'ajouter une seconde source de balises. Vérifier que son intégration Elementor est active (analyse du contenu Elementor, panneau SEO dans l'éditeur).
- Si aucun plugin SEO : recommander explicitement entre plugin dédié et mu-plugin minimal, avec justification. Elementor et Elementor Pro n'émettent pas de balises Open Graph : ne pas les présenter comme une solution.
- Doublons : identifier la source à désactiver (réglage de meta description de Hello Elementor, snippet Elementor Custom Code, widget HTML, addon, second plugin SEO).
- Titres et descriptions des pages Elementor : renseigner les champs SEO dédiés de chaque page dans le plugin SEO plutôt que de compter sur la génération automatique depuis le contenu.
- Image par défaut : format 1200×630, dans la médiathèque, déclarée dans la config du plugin SEO. Recommander une image mise en avant (ou une image sociale dédiée dans le plugin SEO) pour chaque page Elementor.
- URLs incorrectes après migration : proposer `wp search-replace` (avec `--dry-run` et sauvegarde préalable) ou l'outil Elementor de remplacement d'URL, uniquement après validation.
- Mode maintenance Elementor : le désactiver ou restreindre son usage à un environnement non public.
- Cache : inclure la purge nécessaire après correction (plugin de cache, cache hébergeur, CDN/Cloudflare, et régénération des fichiers CSS Elementor si des templates ont été modifiés).
- Vérification post-correction : relancer les phases 3–4 sur les URLs concernées ; recommander le contrôle via le Facebook Sharing Debugger et le LinkedIn Post Inspector (purge du cache des plateformes).

Ne rien implémenter. Présenter le plan et attendre validation.