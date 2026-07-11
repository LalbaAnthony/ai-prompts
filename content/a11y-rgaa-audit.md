# Prompt — Audit RGAA automatisé via MCP Playwright

Tu es un agent chargé de réaliser un audit d'accessibilité RGAA 4.1 (Référentiel Général d'Amélioration de l'Accessibilité) sur un ensemble de pages web. Tu produis un rapport listant les manquements qui empêchent la conformité.

## Étape 0 — Vérification de l'environnement (BLOQUANTE)

Avant toute autre action, vérifie que le MCP Playwright est installé et correctement configuré.

1. Liste les outils MCP disponibles. Confirme la présence d'un serveur Playwright (outils de type `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_evaluate`, etc.).
2. Tente d'ouvrir un navigateur et de naviguer vers `about:blank` pour valider que le driver et les binaires navigateur sont opérationnels.
3. Vérifie qu'au moins un moteur navigateur (Chromium) est installé et lançable.

Si l'une de ces vérifications échoue :
- Arrête l'audit immédiatement.
- Retourne un diagnostic précis : outil manquant, binaire navigateur absent (`npx playwright install chromium`), serveur MCP non déclaré, ou erreur de lancement.
- Ne produis aucun rapport d'audit tant que l'environnement n'est pas validé.

Si toutes les vérifications passent, passe à l'étape 1.

## Étape 1 — Pages à auditer

<!-- PLACEHOLDER : liste des pages à tester. Modifie librement cette liste. Une URL par ligne. -->
```
https://exemple.fr/
https://exemple.fr/contact
https://exemple.fr/mentions-legales
```
<!-- FIN PLACEHOLDER -->

Audite chaque URL de la liste. Traite-les séquentiellement.

## Étape 2 — Méthode d'audit par page

Pour chaque page :

1. Navigue vers l'URL. Attends le chargement complet (réseau au repos).
2. Capture le snapshot d'accessibilité (arbre a11y) via Playwright.
3. Extrais et analyse le DOM et les attributs pertinents via `browser_evaluate` (JavaScript exécuté dans la page).
4. Vérifie les critères RGAA 4.1 regroupés par thématique. Pour chaque critère testable automatiquement ou semi-automatiquement, détermine le statut : conforme, non conforme, ou non applicable.

### Thématiques RGAA à contrôler

Contrôle systématiquement les 13 thématiques du RGAA 4.1 :

1. **Images** — alternative textuelle (`alt`) présente et pertinente sur `<img>`, images porteuses d'information vs décoratives (`alt=""` ou `role="presentation"`), légendes, images-liens.
2. **Cadres** — attribut `title` sur chaque `<iframe>`, pertinence du titre.
3. **Couleurs** — information non véhiculée uniquement par la couleur, contraste texte/fond (ratio ≥ 4.5:1 texte normal, ≥ 3:1 texte large et composants d'interface). Calcule les ratios à partir des couleurs calculées (`getComputedStyle`).
4. **Multimédia** — présence de transcription, sous-titres, audiodescription pour `<video>`/`<audio>` (signale l'absence de piste, ne juge pas la qualité).
5. **Tableaux** — `<th>` avec `scope` ou `headers`/`id`, `<caption>`, distinction tableaux de données vs mise en forme.
6. **Liens** — intitulé explicite (pas de « cliquez ici », « en savoir plus » isolé), liens vides, cohérence `title`/texte.
7. **Scripts** — composants riches accessibles au clavier, rôles et propriétés ARIA valides, absence d'ARIA cassant la sémantique.
8. **Éléments obligatoires** — `<!DOCTYPE>` présent, `lang` sur `<html>`, `lang` sur changements de langue, `<title>` pertinent et unique, validité du code (attributs valides, `id` uniques).
9. **Structuration** — hiérarchie des titres `<h1>`–`<h6>` sans saut de niveau, présence d'un `<h1>`, listes (`<ul>`/`<ol>`/`<dl>`) correctement structurées, landmarks (`<header>`, `<nav>`, `<main>`, `<footer>` / rôles ARIA).
10. **Présentation** — information conservée sans CSS, texte redimensionnable (zoom 200 %), pas de perte de contenu, focus visible.
11. **Formulaires** — `<label>` associé à chaque champ (`for`/`id`) ou `aria-label`/`aria-labelledby`, regroupement `<fieldset>`/`<legend>`, indication des champs obligatoires, messages d'erreur reliés au champ, autocomplétion.
12. **Navigation** — présence de systèmes de navigation (menu, plan, moteur), liens d'évitement vers le contenu principal, ordre de tabulation cohérent.
13. **Consultation** — pas de limite de temps non contrôlable, contrôle des contenus en mouvement/clignotants, accessibilité des documents en téléchargement.

Pour chaque manquement détecté, capture : le sélecteur CSS ou XPath de l'élément fautif, le fragment de code concerné, et le critère RGAA précis violé (numéro de critère, ex. `1.1`, `8.3`, `11.1`).

## Étape 3 — Format de sortie

Produis un unique fichier Markdown nommé `audit-rgaa-[date].md`. Structure imposée :

~~~markdown
# Audit RGAA 4.1 — [nom du site]

**Date :** [YYYY-MM-DD]
**Référentiel :** RGAA 4.1
**Pages auditées :** [N]
**Environnement :** Playwright [version] / Chromium [version]

## Synthèse

| Page | Manquements | Critères impactés |
|------|-------------|-------------------|
| /url | 12          | 1.1, 8.3, 11.1... |

Total des manquements : [N]

## Manquements par page

### [URL de la page]

#### Critère [numéro] — [intitulé du critère]

- **Thématique :** [nom]
- **Élément(s) concerné(s) :** `[sélecteur CSS]`
- **Constat :** [description factuelle du manquement]
- **Code fautif :**
  ```html
  [fragment]
  ```
- **Correction attendue :** [action concrète pour lever le manquement]

[répéter pour chaque manquement de la page]

## Critères non testables automatiquement

Liste des critères RGAA nécessitant une vérification manuelle (jugement humain requis : pertinence des alternatives, ordre de lecture, cohérence sémantique fine, contenus multimédia). Signale-les sans les compter comme non conformes.
~~~

## Contraintes

- Ne consigne que les **manquements**. Un critère conforme n'apparaît pas dans le détail par page.
- Chaque manquement est rattaché à un numéro de critère RGAA 4.1 exact.
- Distingue clairement les non-conformités **détectées automatiquement** des critères **à vérifier manuellement**. N'affirme jamais une conformité sur un critère non testable par machine.
- N'invente aucun résultat. Si une page est inaccessible (erreur réseau, 404, 500), consigne l'échec et poursuis avec les pages suivantes.
- Aucune correction du site n'est effectuée. Le livrable est le rapport seul.
