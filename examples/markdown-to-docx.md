# Mission

Tu dois convertir un document au format **Markdown (.md)** en un document **Microsoft Word (.docx)**.

# Exigences absolues

Le contenu du document est **intouchable**.

Cela signifie notamment que tu ne dois **en aucun cas** :

* ajouter du texte ;
* supprimer du texte ;
* reformuler une phrase ;
* corriger une faute d'orthographe, de grammaire ou de ponctuation ;
* modifier la casse ;
* changer un caractère, un espace, un retour à la ligne, une liste, un tableau, un lien ou un bloc de code autrement que ce qui est strictement nécessaire pour la conversion de format.

Le texte du document Word doit être une représentation fidèle du document Markdown. Le contenu doit être conservé **strictement à l'identique**. Toute modification du contenu est considérée comme une erreur.

La colorisation décrite ci-dessous est une opération **purement visuelle**. Elle ne doit jamais entraîner l'ajout, la suppression ou le déplacement du moindre caractère.

# Mise en forme

La mise en forme est libre d'interprétation, mais doit respecter les principes suivants :

* style sobre et professionnel, proche d'une documentation technique d'entreprise ;
* mise en page simple, sans éléments décoratifs ;
* utilisation des styles Word natifs (Titre 1, Titre 2, Normal, etc.) plutôt que du formatage manuel : la couleur doit être portée par la **définition du style**, pas par du formatage direct, afin de rester modifiable globalement dans Word ;
* hiérarchie des titres conservée ;
* listes, tableaux, citations, liens et blocs de code correctement retranscrits ;
* police standard (par exemple Calibri ou Aptos) ;
* tailles de police classiques ;
* marges standard ;
* espacement raisonnable entre les paragraphes.

L'objectif est d'obtenir un document facilement lisible et **facilement modifiable ultérieurement** dans Microsoft Word.

# Palette de couleurs

La couleur est un outil de lisibilité, jamais de décoration. Elle reste sobre, désaturée, et repose sur une seule dominante bleu foncé avec des gris neutres. Aucune couleur vive, aucun aplat coloré étendu, aucun dégradé.

Palette de référence (à respecter, sauf incohérence manifeste avec le contenu) :

| Élément                     | Couleur                                     | Hex                              |
| --------------------------- | ------------------------------------------- | -------------------------------- |
| Titre 1                     | Bleu nuit                                   | `#1F3864`                        |
| Titre 2                     | Bleu foncé                                  | `#2E5984`                        |
| Titre 3                     | Bleu-gris                                   | `#44688C`                        |
| Titres 4 à 6                | Gris ardoise                                | `#4A5568`                        |
| Corps de texte              | Gris très foncé                             | `#1A1A1A`                        |
| Texte en gras               | Identique au corps                          | `#1A1A1A`                        |
| Liens hypertexte            | Bleu moyen, souligné                        | `#1F5FA9`                        |
| Citations (blockquote)      | Gris moyen, italique, barre latérale gauche | Texte `#4A5568`, barre `#2E5984` |
| Code en ligne (`inline`)    | Bordeaux sur fond gris clair                | Texte `#A31515`, fond `#F2F2F2`  |
| En-tête de tableau          | Texte blanc sur fond bleu foncé             | Texte `#FFFFFF`, fond `#2E5984`  |
| Bordures de tableau         | Gris clair                                  | `#C9D1D9`                        |
| Lignes alternées de tableau | Gris très clair                             | `#F7F9FB`                        |
| Filet horizontal (`---`)    | Gris clair                                  | `#C9D1D9`                        |
| Fond des blocs de code      | Gris très clair                             | `#F6F8FA`                        |
| Bordure des blocs de code   | Gris clair                                  | `#D0D7DE`                        |

Les titres de niveau 1 et 2 peuvent être soulignés d'un filet fin de la même couleur que le titre. Aucun autre effet.

# Blocs de code et coloration syntaxique

Tous les blocs de code doivent être reproduits **strictement à l'identique**, en conservant leur contenu, leur indentation et leurs retours à la ligne. Ils adoptent une police à chasse fixe (Consolas ou Courier New), une taille légèrement réduite (9 à 10 pt), un fond `#F6F8FA` et une fine bordure `#D0D7DE`.

Chaque bloc de code doit être **coloré syntaxiquement en fonction du langage déclaré** après les triples backticks (` ```ts `, ` ```php `, ` ```sql `, ` ```bash `, etc.). La coloration s'applique caractère par caractère, sans jamais altérer le texte lui-même.

Thème de coloration à appliquer (inspiré des thèmes clairs classiques, volontairement désaturé) :

| Jeton                                                                            | Couleur              | Hex       |
| -------------------------------------------------------------------------------- | -------------------- | --------- |
| Texte par défaut / ponctuation                                                   | Gris très foncé      | `#24292F` |
| Mots-clés du langage (`if`, `return`, `function`, `class`, `const`, `SELECT`, …) | Bleu foncé           | `#0033B3` |
| Types, classes, noms de balises                                                  | Sarcelle foncé       | `#1F6F6F` |
| Chaînes de caractères                                                            | Bordeaux             | `#A31515` |
| Nombres et littéraux booléens                                                    | Vert-bleu foncé      | `#0B7285` |
| Commentaires                                                                     | Gris moyen, italique | `#6A737D` |
| Noms de fonctions et de méthodes                                                 | Ocre foncé           | `#7A5C00` |
| Variables, propriétés, attributs                                                 | Bleu-gris            | `#3B5A7A` |
| Opérateurs                                                                       | Gris très foncé      | `#24292F` |
| Directives, décorateurs, annotations, préprocesseur                              | Violet sobre         | `#6B3FA0` |

Règles complémentaires :

* si aucun langage n'est déclaré, ou s'il est inconnu, le bloc reste monochrome en `#24292F` sur fond `#F6F8FA` ;
* le fichier de sortie ne doit contenir aucun surlignage de fond au niveau des jetons individuels : seule la couleur de police varie à l'intérieur du bloc ;
* la coloration ne doit jamais introduire d'espace insécable, de saut de ligne ou de césure supplémentaire ;
* en cas de doute sur l'analyse lexicale d'un fragment, laisser ce fragment en couleur par défaut plutôt que de risquer une altération.

# Schémas Mermaid

Les schémas **Mermaid** ne doivent **jamais** être interprétés, rendus, convertis en image, modifiés ou reformattés. Ils sont recopiés exactement tels qu'ils apparaissent dans le document Markdown, comme n'importe quel autre bloc de code.

Ils ne reçoivent **aucune coloration syntaxique** : texte monochrome `#24292F` sur fond `#F6F8FA`, police à chasse fixe. Cette exception est volontaire et garantit l'absence totale d'altération.

# Images

Si le document Markdown contient des images, elles doivent être intégrées à leur emplacement logique lorsque cela est possible. À défaut, conserver la référence telle quelle sans modifier le reste du contenu.

# Résultat attendu

Produire un unique fichier `.docx` fidèle au document Markdown d'origine.

Le succès de la tâche est évalué selon cet ordre de priorité :

1. préservation intégrale et exacte du texte ;
2. exactitude structurelle (titres, listes, tableaux, blocs de code) ;
3. application de la palette et de la coloration syntaxique.

Une couleur manquante est un défaut mineur. Un caractère modifié est un échec.