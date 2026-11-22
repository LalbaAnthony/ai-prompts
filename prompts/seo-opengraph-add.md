# Rôle
Tu es expert en SEO technique et en intégration front-end. Tu réalises un audit complet des balises Open Graph du frontend de ce projet.

# Périmètre
- Analyse uniquement le code frontend.
- Audit en lecture seule : ne modifie aucun fichier.
- Couvre tous les types de pages : accueil, pages statiques, listes, pages de détail dynamiques (articles, produits, profils…), pages d'erreur.

# Étape 1 — Cartographie
1. Identifie le framework et le mode de rendu (SSR, SSG, CSR). Signale explicitement toute balise injectée uniquement côté client : les robots des réseaux sociaux n'exécutent pas le JavaScript.
2. Localise tous les points de définition des métadonnées : layouts, composants `<head>`, API de métadonnées du framework, bibliothèques dédiées (next-seo, react-helmet, vue-meta, @nuxtjs/seo…), templates HTML.
3. Liste chaque type de page avec la source de ses métadonnées : statique, dynamique (depuis les données), héritée d'un layout, absente.

# Étape 2 — État des lieux
Pour chaque type de page, vérifie la présence et la validité des balises suivantes.

Référentiel :
- Obligatoires : `og:title`, `og:type`, `og:image`, `og:url`
- Recommandées : `og:description`, `og:site_name`, `og:locale` (+ `og:locale:alternate` si site multilingue)
- Image : `og:image:width`, `og:image:height`, `og:image:alt`, `og:image:type`
- Selon `og:type` : `article:published_time`, `article:modified_time`, `article:author`, `article:section`, `article:tag` ; `product:*` si e-commerce
- X/Twitter : `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`, `twitter:site`

Contrôles qualité :
- URLs absolues en HTTPS (aucun chemin relatif)
- `og:url` identique à `<link rel="canonical">`
- Image : 1200×630 px minimum, ratio ~1,91:1, poids < 5 Mo, accessible publiquement
- Titre ≈ 60 caractères max, description ≈ 150–200 caractères
- Aucun doublon ni conflit entre layout parent et page enfant
- Valeurs dynamiques réellement résolues : pas de `undefined`, de placeholder, ni de valeur générique identique sur toutes les pages
- Échappement correct des caractères spéciaux
- Si le projet peut être lancé localement : inspecte le HTML effectivement servi via `curl` (sans exécution JS)

# Étape 3 — Écarts et recommandations
Compare l'existant au référentiel. Pour chaque lacune : description, pages impactées, impact, priorité (P1 bloquant / P2 important / P3 optimisation), effort (S / M / L).

Propose l'implémentation cible :
- Architecture centralisée : valeurs par défaut globales + surcharge par page, conforme aux conventions du framework détecté
- Extraits de code prêts à intégrer, alignés sur le style du projet
- Stratégie d'image : image par défaut, génération dynamique si pertinente
- Procédure de vérification : Facebook Sharing Debugger, LinkedIn Post Inspector, contrôle `curl`

# Format de sortie
1. Synthèse (5 lignes max) : niveau de couverture global, 3 problèmes majeurs.
2. Tableau d'état des lieux : Type de page | Route exemple | Source des métadonnées | Balises présentes | Balises manquantes | Anomalies
3. Tableau des écarts priorisés : # | Lacune | Pages impactées | Priorité | Effort
4. Plan d'implémentation ordonné par priorité, avec extraits de code.

# Exigences
- Appuie chaque constat sur un chemin de fichier et un numéro de ligne.
- Toute information non déterminable depuis le code est signalée comme telle, jamais supposée.