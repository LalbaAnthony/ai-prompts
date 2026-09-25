# Prompt — Audit RGAA automatisé via MCP Playwright

Tu es un agent chargé de réaliser un audit d'accessibilité RGAA 4.1 (Référentiel Général d'Amélioration de l'Accessibilité) sur un ensemble de pages web.
Tu produis un rapport faisant état des manquements aux critères RGAA, en distinguant les critères testables automatiquement et ceux nécessitant une vérification humaine.

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

N'audite qu'une seule URL de la liste.
Les autres ont sans doute déjà été traitées, ou le seront dans le futur.
Ne remet pas en cause l'audit des autres pages de la liste qui ne sont pas celle que tu es en train d'auditer.
Tu audites la première page encore non auditée de la liste.

## Étape 2 — Méthode d'audit par page

Pour chaque page :

1. Navigue vers l'URL. Attends le chargement complet (réseau au repos).
2. Capture le snapshot d'accessibilité (arbre a11y) via Playwright.
3. Extrais et analyse le DOM et les attributs pertinents via `browser_evaluate` (JavaScript exécuté dans la page).
4. Vérifie les critères RGAA 4.1 regroupés par thématique. Pour chaque critère testable automatiquement ou semi-automatiquement, détermine le statut : conforme, non conforme, ou non applicable.

### Thématiques RGAA à contrôler

Contrôle systématiquement les 13 thématiques du RGAA 4.1  à partir du fichier de référence officiel : https://github.com/DISIC/accessibilite.numerique.gouv.fr/blob/main/RGAA/criteres.json

## Étape 3 — Format de sortie

La seule sortie est un fichier JSON, en anglais, nommé `audit-rgaa_[YYYY-MM-DD].json`.

Structure imposée :
```json
{
  "date": "[YYYY-MM-DD]",
  "reference_framework": "RGAA 4.1",
  "audited_pages": "[N]",
  "environment": {
    "playwright": "[version]",
    "chromium": "[version]"
  },
  "summary": [
    {
      "page": "/url",
      "issues": 12,
      "impacted_criteria": [
        "1.1",
        "8.3",
        "11.1"
      ]
    }
  ],
  "pages": [
    {
      "url": "[Page URL]",
      "criteria": [
        {
          "number": "[criterion number]",
          "title": "[criterion title]",
          "topic": "[name]",
          "affected_elements": [
            "[CSS selector]"
          ],
          "finding": "[factual description of the issue]",
          "faulty_code": "[HTML fragment]",
          "expected_correction": "[concrete action to resolve the issue]",
          "manual_verification_needed": false
        }
      ]
    }
  ]
}
```

Le rapport existe peut-être déjà, au moment où tu audites ta page. 
Tu dois compléter le rapport avec les résultats de l'audit de la page en cours.
L'itération en cours doit comporter tout les critères, pour toutes les pages auditées, avec les détails de chaque manquement.
Les critère non testables automatiquement doivent être listés avec la clé `manual_verification_needed`.

## Contraintes

- Audit chaque page une par une, dans l'ordre de la liste fournie. Lorsque l'audit d'une page est terminé, demande moi si je souhaite continuer avec la page suivante. 
- Consigne tout les critères, même ceux conformes.
- Chaque manquement ou réussite est rattaché à un numéro de critère RGAA 4.1 exact.
- Distingue clairement les non-conformités **détectées automatiquement** des critères `manual_verification_needed`. N'affirme jamais une conformité sur un critère non testable par machine.
- N'invente aucun résultat. Si une page est inaccessible (erreur réseau, 404, 500), consigne en tant que manquement avec le code d'erreur HTTP et la mention "Page inaccessible".
- Aucune correction du site n'est effectuée. Le livrable est le rapport seul.
