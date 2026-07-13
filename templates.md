# Templates

## Squelette

Copier le bloc, remplacer chaque `[...]`, supprimer les balises inutilisées.

~~~md
<role>
[Expertise incarnée. Ex : "Tu es un ingénieur data senior spécialisé en pipelines temps réel."
Le rôle fixe le niveau technique et le vocabulaire.]
</role>

<contexte>
[Situation, audience cible, finalité, ce qui est déjà connu ou décidé, contraintes de fond
(budget, stack, délai, ton attendu du livrable).]
</contexte>

<objectif>
[Le livrable unique et précis : verbe d'action + artefact exact + résultat attendu.
Ex : "Produis une fonction Python qui déduplique une liste de dicts sur la clé 'id',
en conservant l'occurrence la plus récente."]
</objectif>

<instructions>
- [Règle 1 : ce qu'il faut faire]
- [Règle 2 : ordre, priorités, méthode]
- [À exclure explicitement : ...]
- [Traitement de l'incertitude : "Si une information manque, énonce tes hypothèses
  avant de produire, ne devine pas silencieusement."]
</instructions>

<donnees>
[Matériau brut à traiter : texte source, code existant, données, fichier.
Isolé ici pour ne pas être lu comme une instruction.]
</donnees>

<exemples>
Entrée : [...]
Sortie attendue : [...]

Contre-exemple — NE PAS produire ceci : [...]
Raison : [...]
</exemples>

<raisonnement>
[Réfléchis étape par étape dans un bloc <analyse> avant de répondre.
Ne montre ce raisonnement que si explicitement demandé.]
</raisonnement>

<format>
[Structure exacte : titres, longueur (ex : 300 mots max), langue, ton,
schéma JSON ou gabarit si applicable.]
</format>

<criteres>
[Conditions de validation. "La réponse est correcte si et seulement si : ...".
Répéter ici la contrainte la plus critique.]
</criteres>
~~~

## Exemple rempli

~~~md
<role>
Tu es un rédacteur technique qui écrit pour des développeurs pressés.
</role>

<contexte>
Documentation interne d'une API REST de paiement. Lecteurs : intégrateurs
back-end découvrant l'API. Objectif : qu'ils réussissent un premier appel
en moins de cinq minutes.
</contexte>

<objectif>
Rédige la section "Premier appel" du guide de démarrage.
</objectif>

<instructions>
- Commence par le prérequis minimal (clé API).
- Donne un exemple de requête curl complet et exécutable.
- Montre la réponse de succès attendue.
- Liste les deux erreurs les plus fréquentes et leur correction.
- N'explique pas l'architecture interne de l'API.
- Si un paramètre est ambigu, signale-le au lieu d'inventer une valeur.
</instructions>

<format>
Markdown. Titres de niveau 3. 250 mots maximum hors blocs de code.
Blocs de code annotés avec le langage.
</format>

<criteres>
La section est correcte si un développeur peut copier-coller l'exemple,
l'exécuter, et obtenir une réponse 200 sans lire d'autre documentation.
</criteres>
~~~

## Règles de rédaction par section

- **role** : Optionnel mais puissant. À utiliser quand le niveau d'expertise ou le vocabulaire change la qualité. Inutile pour une tâche factuelle simple.
- **contexte** : La cause n°1 de réponses médiocres est un contexte absent, pas un modèle limité. Charger tout ce qu'un exécutant humain compétent exigerait avant de - commencer.
- **objectif** : Un seul. Si plusieurs livrables sont nécessaires, les numéroter et fixer leur ordre. Ambiguïté sur le livrable = échec garanti.
- **instructions** : Formuler en positif (« fais X ») plutôt qu'en négatif seul. Ajouter les exclusions à part. Toujours inclure la clause de gestion de l'incertitude.
- **donnees** : Toute entrée volumineuse va ici, jamais mélangée aux consignes. Les balises empêchent l'injection de consignes parasites contenues dans les données.
- **exemples** : Un à trois. Le contre-exemple vaut souvent plus que l'exemple : il ferme une porte d'erreur précise.
- **raisonnement** : « étape par étape » améliore l'exactitude sur le calcul, la logique, l'analyse. Distinguer le raisonnement (interne) de la réponse (livrée).
- **format** : Spécifier longueur, structure, langue. Pour une sortie machine, imposer un schéma strict et interdire tout texte hors schéma.
- **criteres** : Transforme un prompt vague en spécification testable. Permet au modèle d'auto-vérifier avant de rendre.

## Checklist avant envoi

- [ ] Le livrable est unique, nommé, borné.
- [ ] Instructions et données sont séparées par des balises.
- [ ] Le contexte suffit à un exécutant qui ne connaît pas le projet.
- [ ] Au moins un exemple ; un contre-exemple si la limite est floue.
- [ ] Le raisonnement étape par étape est exigé pour toute tâche non triviale.
- [ ] Format, longueur, langue sont imposés.
- [ ] Les critères de réussite sont testables.
- [ ] La gestion de l'information manquante est spécifiée.
- [ ] La contrainte critique est répétée en fin de prompt.