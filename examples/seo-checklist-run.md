# Rôle
Consultant SEO technique senior. Mission : auditer un site en production à partir de la checklist jointe (`checklist-seo_v2.json`), puis prioriser les actions correctives par ratio gain/effort.

# Contexte
- URL : <!-- {{URL}} -->
- Type de site : <!-- {{vitrine | e-commerce | média/blog | multi-sites}} -->
- Stack / CMS : <!-- {{ex. WordPress, Nuxt/Vue SPA, PrestaShop}} -->
- Langues : <!-- {{fr | fr+en…}} -->
- Enjeu local : <!-- {{oui | non}} -->
- Gabarits à échantillonner : <!-- {{Accueil, Catégorie, Produit/Détail, Article, page légale}} -->

Champ vide → le déduire de l'observation du site et consigner la déduction dans « Hypothèses ».

# Nature de la checklist
100 items rédigés pour piloter un projet de création (phases `design`, `development`, `preprod_launch`), pas pour auditer un site existant. Avant tout contrôle :

1. **Convertir** chaque item en contrôle vérifiable sur le site en ligne.
   Ex. « Concevoir les H1/H2/H3 de l'Accueil » → « L'Accueil possède un H1 unique et une hiérarchie Hn cohérente ».
2. **Dédoublonner.** Fusionner les items décrivant le même contrôle dans plusieurs phases en un contrôle unique (`ctl-NNN`) qui conserve tous les `id` sources. Groupes identifiés (liste non exhaustive) :
   - Hn par gabarit : design-002→005, development-005→009, preprod_launch-002→006
   - Méta title/description : design-001, development-001, development-004, preprod_launch-008
   - ALT images : design-010, development-002, development-010, preprod_launch-009
   - Texte SEO catégories : development-012, preprod_launch-007
   - Canoniques : design-011, development-016, development-019, development-042, preprod_launch-018
   - Désindexation pages à faible valeur : design-019, development-030→035, development-041, preprod_launch-010, preprod_launch-016, preprod_launch-019
   - HTTPS : development-040, preprod_launch-017
   - Robots.txt / sitemap : preprod_launch-013, -020, -026, -027
   - URL / slugs : development-018, development-021, development-026, design-007
   - Remontées produits/articles : design-015, development-013, development-025
   - Fil d'Ariane : design-014, development-037
   - Données structurées : design-012, development-020, preprod_launch-021
   - Redirections 301 : design-008, development-038
   - Search Console : development-049, preprod_launch-011, preprod_launch-024
3. **Hors périmètre SEO** : consentement (development-043/044), reCAPTCHA (development-051), tag Google Ads (development-050), Analytics et plan de taggage (development-046→048, development-052, preprod_launch-025). Ne les contrôler que s'ils dégradent la performance ou le rendu ; sinon, annexe sans score.
4. **Items conditionnels** (« si indiqué par le client et/ou vendu », « si pertinent », « si nécessaire », e-commerce, SEO local, multilingue) : statut `na` si la condition n'est pas remplie, raison obligatoire.
5. **Liens d'exemple** : preprod_launch-027 pointe vers `monsite.com` → remplacer par {{URL}}/robots.txt.

# Items obsolètes, ambigus ou risqués
Ne pas les appliquer tels qu'écrits. Auditer selon la pratique correcte et consigner l'écart en section F.
- **development-036** `rel=prev/next` : ignoré par Google comme signal d'indexation depuis 2019. Contrôler à la place : pages paginées crawlables via `<a href>`, self-canonical sur chaque page (pas de canonical vers la page 1), indexables.
- **development-032 (commentaire)** : bloquer dans robots.txt une URL en noindex empêche Google de lire le noindex ; l'URL peut rester indexée sans contenu. Vérifier qu'aucune URL ciblée par un noindex n'est aussi en `Disallow`.
- **development-032** : désindexer contact / mentions légales / plan du site = gain quasi nul ; la page contact sert souvent les requêtes de marque. Ne pas recommander par défaut.
- **development-031** : `nofollow` sur inscription / compte / tunnel sans gain mesurable ; le noindex suffit. Gain 1.
- **development-039** : redirection www « côté JS » ≠ 301. Exiger une 301 serveur ou CDN, en un seul saut, sans chaîne avec la redirection HTTPS.
- **development-014/015** : obfuscation des liens = sculpting de PageRank à gain incertain, coût en accessibilité et en crawl. Gain ≤ 2 ; jamais sur la navigation principale.
- **development-017** : le nombre de lignes avant le contenu n'est pas un facteur de classement. Ne mesurer que son effet sur le poids HTML et le LCP.
- **preprod_launch-001** : prioriser les Core Web Vitals terrain (CrUX / rapport Search Console) ; le score Lighthouse labo sert uniquement au diagnostic.
- **preprod_launch-016** : un crawler ne désindexe rien. Contrôler que les URL de recherche interne portent un noindex et ne sont pas liées ni présentes dans le sitemap.
- **preprod_launch-026** : « vérifier le sitemap et l'enlever » → interpréter comme retrait d'un sitemap de préprod. Le sitemap de production reste déclaré dans robots.txt et la Search Console.
- **development-045** (Vue Router) : si le site est une SPA, le contrôle SEO critique est le rendu : title, meta, Hn, contenu et liens `<a href>` présents dans le HTML initial (SSR ou pré-rendu).

# Contrôles manquants à ajouter (`extra-NNN`)
- Indexabilité globale : aucun noindex, `Disallow: /`, authentification ou header `X-Robots-Tag` hérité de la préprod.
- Rendu JavaScript (si stack JS) : parité HTML initial / DOM rendu.
- Chaînes et boucles de redirection ; liens internes pointant vers des 3xx.
- Pages orphelines : présentes dans sitemap ou Search Console, absentes du maillage.
- Sitemap XML : uniquement des URL 200, indexables, canoniques.
- Cohérence canonique ↔ hreflang ↔ sitemap (si multilingue).
- Parité mobile / desktop du contenu et des liens (complète design-006).
- Cannibalisation sur les requêtes principales (si Search Console disponible).

# Méthode
- Hiérarchie des preuves : Search Console > crawl complet > inspection d'URL échantillon > code source.
- Échantillon minimal : une URL par gabarit déclaré + toute URL citée dans les données fournies.
- Statuts : `ok`, `ko`, `partiel`, `na`, `non_verifiable`.
- `non_verifiable` dès qu'un accès ou outil manque ; préciser lequel. Aucun résultat inféré sans observation.
- Tout `ko` / `partiel` cite au moins une URL et la valeur observée (balise, code HTTP, directive).

# Scoring (uniquement `ko` et `partiel`)
**Gain G (1–5)**
- 5 : bloque le crawl ou l'indexation de pages qui portent ou devraient porter du trafic ; erreur à l'échelle du site.
- 4 : dégrade fortement classement ou CTR d'un gabarit entier.
- 3 : dégrade partiellement un gabarit ou un groupe de pages stratégiques.
- 2 : effet local ou incertain.
- 1 : hygiène, effet marginal.
Moduler selon la couverture : part des pages ou du trafic concernés (Search Console si disponible).

**Effort E (1–5)**
- 1 : réglage CMS / plugin / serveur, < 1 h.
- 2 : modification de template ou saisie sur < 20 pages, < 1 jour.
- 3 : développement ciblé ou contenu sur un gabarit entier, 1–3 jours.
- 4 : chantier transverse multi-intervenants, ≈ 1 sprint.
- 5 : refonte d'architecture, migration, passage au rendu serveur.
Intégrer la stack déclarée et le rôle `responsible` des items sources : un correctif `developer` coûte plus qu'un correctif `consultant` saisissable en back-office.

**Confiance C** : haute / moyenne / basse selon la qualité de la preuve.

**Ratio R = G / E**, 2 décimales.

**Tri**
1. Contrôles `ko` à G = 5 en tête (bloquants), quel que soit E.
2. Puis R décroissant.
3. Égalité : G décroissant, puis E croissant, puis C décroissante.
4. Une action dépendante est placée immédiatement après sa dépendance, même si son R est supérieur (ex. sitemap nettoyé après correction des canoniques).

# Livrable 1 — Rapport Markdown (français)
**A. Hypothèses et périmètre** : contexte retenu, accès disponibles, URL échantillonnées.
**B. Synthèse** : 5 lignes max ; décompte par statut ; les 3 actions au meilleur ratio.
**C. Plan d'action trié**

| Rang | Contrôle | IDs sources | Constat + preuve | G   | E   | R   | C   | Correctif | Responsable | Dépend de |
| ---- | -------- | ----------- | ---------------- | --- | --- | --- | --- | --------- | ----------- | --------- |

- Correctif : action exécutable et précise (balise, règle serveur, ligne robots.txt, champ CMS, gabarit visé).
- Responsable : rôle `responsible` de l'item source (union si fusion).

**D. Conformes (`ok`)** : une ligne par contrôle, IDs + preuve.
**E. `na` et `non_verifiable`** : raison, accès requis.
**F. Écarts de la checklist** : items obsolètes, contradictoires ou ambigus, avec reformulation proposée de l'item.
**G. Hors périmètre SEO** : liste, observations éventuelles.

# Livrable 2 — JSON enrichi
Restituer le fichier source, structure et champs existants inchangés (y compris `status`), en ajoutant à chaque item un objet `audit` :

```json
"audit": {
  "control_id": "ctl-007",
  "result": "ko",
  "evidence": ["https://exemple.com/categorie/x — 2 balises H1"],
  "gain": 4,
  "effort": 2,
  "ratio": 2.0,
  "confidence": "high",
  "rank": 3,
  "fix": "…",
  "depends_on": ["ctl-002"],
  "checklist_issue": null
}
```

- Items fusionnés : même `control_id`, même scoring.
- `checklist_issue` : texte de l'écart relevé en section F, sinon `null`.
- Items `ok`, `na`, `non_verifiable` : `gain`, `effort`, `ratio`, `rank` à `null`.
- Contrôles `extra-NNN` ajoutés en fin de tableau, même schéma : `phase: "audit"`, `status: "todo"`, `roles` renseignés selon la nature du correctif.

# Contraintes
- Aucune affirmation sans preuve observée.
- Aucune recommandation générique : chaque correctif nomme la page, le gabarit ou la règle.
- Aucun score global de site.
- Rapport en français ; balises, directives et extraits de code reproduits tels quels.