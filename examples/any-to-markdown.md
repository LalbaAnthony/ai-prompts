# Mission

Tu dois convertir un ou plusieurs fichiers, **quel que soit leur format** (DOCX, PDF, PPTX, XLSX, CSV, TXT, JSON, YAML, XML, HTML, Markdown ou autre), en documents **Markdown (.md)**.

Les fichiers produits servent à deux usages : la lecture humaine (VS Code, GitHub, Obsidian) et l'exploitation par un agent LLM chargé d'agréger ensuite leur contenu. La structure Markdown est laissée à ton interprétation ; le contenu, lui, doit être reproduit **scrupuleusement à l'identique**.

# Exigences absolues

Le contenu des fichiers source est **intouchable**.

Cela signifie notamment que tu ne dois **en aucun cas** :

* ajouter du texte ;
* supprimer du texte ;
* reformuler, résumer, abréger ou tronquer ;
* traduire ;
* corriger une faute d'orthographe, de grammaire, de ponctuation ou de typographie ;
* modifier la casse, hors règle « Casse » ci-dessous ;
* réordonner le contenu, sauf pour restituer l'ordre de lecture logique d'une mise en page en colonnes ;
* dédupliquer un contenu répété, hors règle « En-têtes, pieds de page, filigranes et numéros de page » ci-dessous ;
* inventer un titre, une légende, un en-tête de colonne, une description d'image ou un texte de remplacement ;
* reformater une valeur : nombre, date, montant, identifiant, URL, code.

Seules les transformations listées dans la section **Transformations de mise en page autorisées** sont permises. Toute autre modification du contenu est une erreur.

Un fichier volumineux est converti **intégralement**. Toute ellipse ajoutée (« … », « [suite identique] », « etc. », « [tronqué] ») est interdite.

# Référence de fidélité

La fidélité se mesure sur le **texte rendu** : le texte affiché par un moteur Markdown (GitHub Flavored Markdown) à partir du fichier produit doit être identique au texte visible dans la source, aux seules transformations autorisées près. Les marqueurs réservés (commentaires HTML, voir **Marqueurs réservés**) ne font pas partie du texte rendu.

## Échappement minimal

Un caractère de la source qui serait interprété comme syntaxe Markdown est précédé d'une barre oblique inverse, **uniquement là où il serait interprété**. Partout ailleurs, il est écrit tel quel : `snake_case`, `a * b` ou `#hashtag` ne sont pas échappés.

| Cas                                                             | Texte source                | Écriture Markdown            |
| --------------------------------------------------------------- | --------------------------- | ---------------------------- |
| Astérisques ou soulignés pouvant former une emphase             | `2*3*4`                     | `2\*3\*4`                    |
| `#` suivi d'un espace en début de ligne                         | `# 1 du classement`         | `\# 1 du classement`         |
| Nombre suivi d'un point en début de ligne, hors liste           | `2024. Une année charnière` | `2024\. Une année charnière` |
| `-`, `+` ou `*` suivi d'un espace en début de ligne, hors liste | `- 5 °C la nuit`            | `\- 5 °C la nuit`            |
| `>` en début de ligne                                           | `> 10 % des cas`            | `\> 10 % des cas`            |
| Crochets suivis de parenthèses                                  | `[A](B)`                    | `\[A\](B)`                   |
| Chevron pouvant ouvrir une balise ou un lien automatique        | `<div>`                     | `\<div>`                     |
| Esperluette pouvant former une entité                           | `&eacute;`                  | `\&eacute;`                  |
| Double tilde                                                    | `~~`                        | `\~\~`                       |
| Ligne isolée de `-`, `=` ou `*` qui n'est pas une décoration    | `---`                       | `\---`                       |
| Dollar, dans tous les cas (la syntaxe mathématique est active)  | `10 $`                      | `10 \$`                      |

Règles complémentaires :

* un accent grave littéral est précédé d'une barre oblique inverse ;
* une barre verticale dans une cellule de tableau Markdown est écrite `\|` ;
* une barre oblique inverse littérale suivie d'un caractère de ponctuation est doublée ;
* aucun échappement dans un bloc de code ni dans un code en ligne ;
* dans un bloc HTML (tableau HTML), le Markdown n'est pas interprété : la mise en forme y est écrite en balises HTML et les caractères `<`, `>`, `&` en entités `&lt;`, `&gt;`, `&amp;`.

# Transformations de mise en page autorisées

Cette liste est exhaustive. Ce qui n'y figure pas est interdit.

## 1. Retours à la ligne de mise en page (reflow)

* Les lignes d'un même paragraphe coupées par la mise en page (PDF, TXT à largeur fixe manifeste) sont rejointes par un seul espace.
* Tiret en fin de ligne :
  * césure syllabique certaine (« conver-⏎sion ») : tiret retiré, parties rejointes sans espace (« conversion ») ;
  * mot composé, nom propre, identifiant ou moindre doute (« porte-⏎monnaie », « peut-⏎être », « Jean-⏎Pierre », « ISO-⏎8859 ») : tiret conservé, parties rejointes sans espace (« porte-monnaie »). Chaque cas douteux est signalé dans le rapport.
* Un titre coupé sur plusieurs lignes par la mise en page est rejoint sur une seule ligne.
* Les retours à la ligne signifiants (adresse, vers, signature, suite de lignes courtes, contenu de cellule) sont conservés en retour à la ligne forcé : barre oblique inverse en fin de ligne, `<br>` dans une cellule de tableau.

## 2. En-têtes, pieds de page, filigranes et numéros de page

* Le texte d'en-tête ou de pied de page (défini comme tel dans un DOCX, répété au même emplacement sur les pages d'un PDF, ou issu du masque d'une présentation) est reproduit **une seule fois** :
  * en-têtes au début du fichier, entre `<!-- PAGE-HEADER:START -->` et `<!-- PAGE-HEADER:END -->` ;
  * pieds de page à la fin du corps, entre `<!-- PAGE-FOOTER:START -->` et `<!-- PAGE-FOOTER:END -->`.
* Plusieurs variantes distinctes (première page, pages paires et impaires, sections) : chacune une fois, dans l'ordre d'apparition.
* Un filigrane textuel (« CONFIDENTIEL », « BROUILLON ») et une image répétée en en-tête (logo) suivent la même règle, entre les marqueurs d'en-tête.
* Numéros de page et de diapositive, compteurs « Page X / Y » ou « X/Y » : supprimés, y compris à l'intérieur d'un en-tête ou d'un pied de page.
* Les renvois de page dans le corps du texte (« voir page 12 ») et la table des matières ne sont pas concernés : ils sont du contenu.

## 3. Puces

* Un glyphe servant uniquement de marqueur de liste en début d'élément (•, ◦, ▪, ▫, ■, ●, ○, ‣, ⁃, –, —, -, *, ➢, ►, →, etc.) est remplacé par le marqueur Markdown `-` ; l'imbrication est rendue par l'indentation.
* Un glyphe porteur de sens (☐ et ☑, ✓ et ✗ alternant dans une même liste, symboles d'état) est du contenu : il est conservé après le marqueur (`- ✓ Validé`).

## 4. Numérotation automatique

* Les numéros générés par le logiciel source (listes numérotées, titres numérotés, légendes « Figure 3 », « Tableau 2 ») sont **matérialisés tels qu'affichés** : « 2.3 Contexte ».
* Liste ordonnée Markdown (`1.`) uniquement si la source affiche une numérotation arabe consécutive suivie d'un point ; le numéro de départ est conservé (`3.` pour une liste qui commence à 3).
* Tout autre style (`1)`, `a.`, `a)`, `i.`, `I.`, `1.1`, numérotation non consécutive) : numéro écrit littéralement en début d'élément, échappé si nécessaire, éléments sur des lignes ou paragraphes distincts, sans puce ajoutée.
* Un numéro saisi à la main dans le texte est du contenu et suit les mêmes règles d'écriture.

## 5. Casse

* Texte affiché en majuscules par un attribut de mise en forme (Majuscules, All caps) : écrit en majuscules, tel qu'affiché.
* Texte en petites capitales : écrit dans sa casse saisie.
* Toute autre casse : inchangée.

## 6. Blancs de mise en page

* Lignes alignées par tabulations ou espaces formant une grille évidente (plusieurs lignes, colonnes constantes) : converties en tableau.
* Autres suites d'espaces ou de tabulations à l'intérieur d'une ligne : conservées brutes dans la source (le rendu les réduit à un espace).
* Retrait de début de ligne : retiré lorsqu'il créerait un bloc de code (quatre espaces ou une tabulation) ou une imbrication involontaire ; un retrait signifiant de liste est rendu par l'indentation Markdown.
* Espaces en fin de ligne : supprimés (deux espaces finaux créeraient un retour forcé invisible).

## 7. Lignes décoratives des fichiers texte brut

* Ligne composée uniquement de `=` ou de `-` soulignant une ligne de texte : la ligne de texte devient un titre (niveau selon la hiérarchie du fichier), le soulignement disparaît.
* Ligne composée d'un même caractère répété (`-`, `=`, `*`, `_`, `~`, `#`, `─`, `═`) servant de séparateur : filet `---`.
* Tableau dessiné en caractères (`+---+`, `│`, `┌`) analysable sans ambiguïté : converti en tableau ; sinon, traité comme un dessin.
* Dessin en caractères (cadre, schéma, arborescence, art ASCII) : bloc de code ` ```text ` verbatim.

## 8. Lignes vides et sauts

* Plusieurs lignes vides consécutives : une seule.
* Paragraphes vides, sauts de page, de section et de colonne : non représentés. Les changements de page PDF sont repérés par le marqueur `PAGE`.

## 9. Caractères invisibles et ligatures

* Tout caractère visible est conservé à l'identique : apostrophes et guillemets typographiques (’ « » “ ”), tirets (– —), points de suspension (…), espaces insécables (U+00A0, U+202F), symboles, emojis.
* Ligatures de présentation (U+FB00 à U+FB06 : ﬀ ﬁ ﬂ ﬃ ﬄ ﬅ ﬆ) : décomposées en lettres (ﬁ → fi).
* Supprimés : trait d'union conditionnel (U+00AD), espace de largeur nulle (U+200B), gluon de mots (U+2060), U+FEFF, caractères de contrôle autres que la tabulation et le saut de ligne.
* Conservés : U+200C et U+200D (rôle linguistique ou emoji).
* Normalisation Unicode NFC (sans effet visible).

## 10. Syntaxe des formats de données

Les caractères purement syntaxiques d'un format de données interprété (accolades, crochets, guillemets délimiteurs et virgules en JSON ; chevrons et guillemets d'attributs en XML ; délimiteurs et guillemets d'encadrement en CSV) disparaissent au profit de la structure Markdown. Les clés, noms, valeurs et textes qu'ils délimitent sont du contenu et restent intacts. Les séquences d'échappement (`\n`, `é`, `&amp;`) sont décodées : c'est le caractère représenté qui est du contenu.

# Méthode d'extraction

## Avec exécution de code (mode nominal)

* L'extraction est **programmatique**. Ne jamais retaper, recopier de mémoire ou reformuler un contenu lu.
* Chaque chaîne écrite dans le .md provient d'une extraction par programme, jamais d'une saisie.
* L'encodage de chaque fichier texte est détecté (UTF-8, UTF-16, Windows-1252, ISO-8859-1 ou 15, Mac Roman…). Un texte mal décodé (« Ã© » au lieu de « é ») est une erreur de conversion ; un texte déjà corrompu dans la source est du contenu et reste tel quel.
* Avant conversion, chaque fichier est vérifié : ouverture possible, chiffrement, corruption, présence d'une couche texte (PDF).
* Les images sont extraites sous leur forme binaire d'origine, sans recompression.

Outils indicatifs, à adapter à l'environnement (installer ce qui manque) :

| Format                            | Outils                                                                                                                                                                                                                                                         |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DOCX                              | pandoc (`-f docx -t gfm --track-changes=accept --extract-media=…`) comme base ; lecture directe du XML (`word/document.xml`, `numbering.xml`, `comments.xml`, `footnotes.xml`, `endnotes.xml`, `header*.xml`, `footer*.xml`) pour ce que pandoc perd ou altère |
| DOC, RTF, ODT, XLS, ODS, PPT, ODP | LibreOffice headless (`soffice --headless --convert-to docx`, `xlsx` ou `pptx`), puis règles du format obtenu                                                                                                                                                  |
| PDF                               | PyMuPDF (texte, positions, polices, images, annotations, champs), pdfplumber (tableaux), `pdftotext -layout` en contrôle croisé                                                                                                                                |
| PPTX                              | python-pptx, lecture directe du XML (`ppt/slides`, `ppt/notesSlides`, `ppt/comments`)                                                                                                                                                                          |
| XLSX                              | openpyxl (feuilles, fusions, masquage, commentaires, valeurs en cache) ; valeurs affichées par l'export CSV « tel qu'affiché » de LibreOffice ou par application des formats de nombre                                                                         |
| CSV, TSV                          | module `csv` (détection du délimiteur), charset-normalizer (encodage)                                                                                                                                                                                          |
| JSON                              | `json` avec `object_pairs_hook` (ordre et clés dupliquées) et `parse_int=str`, `parse_float=str` (forme lexicale des nombres)                                                                                                                                  |
| XML                               | lxml (`remove_comments=False`, `strip_cdata=False`)                                                                                                                                                                                                            |
| HTML                              | pandoc (`-f html -t gfm`) ou BeautifulSoup                                                                                                                                                                                                                     |
| Équations Word                    | pandoc (OMML vers LaTeX)                                                                                                                                                                                                                                       |

## Sans exécution de code (repli)

* Lecture directe du fichier et transcription caractère par caractère, sans aucune reformulation.
* La réponse commence par un avertissement explicite : extraction non programmatique, fidélité non vérifiable mécaniquement.
* Images non extractibles : `<!-- MISSING: image N, non extraite (pas d'exécution de code) -->` à leur emplacement.
* Si la création de fichiers est impossible, chaque .md est livré dans la réponse dans un bloc de code ` ````markdown ` dont la clôture est plus longue que toute suite d'accents graves du contenu, précédé du nom du fichier de sortie, hors du bloc.
* Si la longueur maximale de réponse est atteinte, la suite est livrée dans le message suivant, exactement à partir du caractère où le précédent s'est arrêté, sans rien abréger ni répéter.

# Fichiers de sortie

* Un fichier .md par fichier source, sans agrégation ni contenu croisé entre fichiers.
* Nom : `<nom>.<ext>.md` (`rapport.pdf` → `rapport.pdf.md`, `rapport.docx` → `rapport.docx.md`).
* Images extraites : dossier `<nom>.<ext>.assets/`, fichiers `image-001.<ext>`, `image-002.<ext>`… dans l'ordre d'apparition, format d'origine. Dossier créé uniquement s'il y a des images.
* Format d'image non affichable par un navigateur (EMF, WMF, TIFF) : original conservé, plus une copie PNG vers laquelle pointe le lien.
* Liens relatifs ; chemin contenant des espaces ou des parenthèses encadré de chevrons : `![](<rapport final.pdf.assets/image-001.png>)`.
* Encodage UTF-8 sans BOM, fins de ligne LF, saut de ligne final.
* Syntaxe : GitHub Flavored Markdown, avec notes `[^n]` et mathématiques `$…$` / `$$…$$`.
* Première ligne : `<!-- SOURCE: <nom du fichier source> -->` (après le front matter éventuel d'un fichier Markdown source).

# Conventions Markdown

## Titres

* Déduits de la source : styles de titre (Titre, Titre 1 à 6), taille et graisse de police et signets (PDF), titres de diapositive, noms de feuille, soulignements (TXT).
* Le texte du titre est celui de la source, numérotation matérialisée comprise. Aucun titre n'est inventé : un contenu sans titre reste sans titre.
* La hiérarchie relative est conservée, sur les niveaux 1 à 6.

## Paragraphes et retours forcés

* Paragraphes séparés par une ligne vide.
* Retour à la ligne forcé : barre oblique inverse en fin de ligne ; `<br>` dans une cellule de tableau.

## Mise en forme des caractères

| Source                                          | Markdown           |
| ----------------------------------------------- | ------------------ |
| Gras                                            | `**texte**`        |
| Italique                                        | `*texte*`          |
| Gras italique                                   | `***texte***`      |
| Barré                                           | `~~texte~~`        |
| Souligné                                        | `<u>texte</u>`     |
| Exposant                                        | `<sup>texte</sup>` |
| Indice                                          | `<sub>texte</sub>` |
| Police à chasse fixe dans le texte courant      | `` `texte` ``      |
| Couleur, surlignage, police, taille, espacement | ignorés            |

* Les espaces en bordure d'un passage mis en forme sont placés à l'extérieur des marqueurs (`**gras** suite`, jamais `**gras **suite`), sans être ni supprimés ni ajoutés.
* Emphase à l'intérieur d'un mot : astérisques, jamais soulignés.

## Liens

* `[texte](url)`, URL recopiée à l'identique ; URL contenant des espaces ou des parenthèses : destination entre chevrons, `[texte](<url>)`.
* Texte identique à l'URL : lien automatique `<url>`.
* Infobulle présente dans la source : `[texte](url "infobulle")`.
* Lien interne vers un signet ou une page du document source (table des matières, renvois) : texte conservé, sans lien.

## Tableaux

* Tableau simple (ligne d'en-tête identifiable, cellules d'une seule ligne de texte, aucune fusion, aucun masquage) : tableau GFM.
* Tableau complexe (cellules fusionnées, plusieurs paragraphes ou liste dans une cellule, tableau imbriqué, absence de ligne d'en-tête, lignes ou colonnes masquées) : tableau HTML `<table>` avec `rowspan` et `colspan`, `<thead>` seulement si une ligne d'en-tête existe, aucun attribut de style.
* La valeur d'une cellule fusionnée n'est écrite qu'une fois ; aucune valeur n'est dupliquée, aucun en-tête de colonne n'est inventé.
* Cellule vide : vide.
* Légende de tableau présente dans la source : paragraphe à son emplacement.

## Notes de bas et de fin de page

* Appel `[^n]` à l'emplacement de l'appel source ; définition `[^n]: texte` en fin de fichier, dans l'ordre. L'étiquette reprend le numéro ou le symbole d'appel de la source.
* Note de plusieurs paragraphes : paragraphes suivants indentés de quatre espaces.

## Équations

* Équation native (Word, PowerPoint) : LaTeX, `$…$` en ligne, `$$…$$` en bloc.
* Formule dans un PDF : LaTeX uniquement si sa reconstruction est certaine ; sinon `<!-- MISSING: formule non reconstructible, caractères extraits : … -->`, signalé dans le rapport.

## Code

* Paragraphes en police à chasse fixe ou de style code (« Code », « HTML préformaté », Consolas, Courier) : bloc de code clôturé, contenu verbatim, indentation et retours conservés, aucun échappement.
* Étiquette de langage uniquement si le langage est déclaré dans la source ou sans ambiguïté.
* Clôture plus longue que la plus longue suite d'accents graves du contenu.

## Images

* Image extraite dans le dossier d'images et référencée à son emplacement logique : `![alt](chemin)`.
* `alt` : texte alternatif de la source s'il existe, sinon vide (`![](chemin)`). Aucune description rédigée.
* Légende présente dans la source (« Figure 2 : … ») : paragraphe à son emplacement, comme tout texte.
* Image distante (HTML) : URL d'origine conservée, aucun téléchargement.

## Graphiques natifs

* Graphique Excel, Word ou PowerPoint (objet graphique, pas une image) : à son emplacement, entre `<!-- CHART:START -->` et `<!-- CHART:END -->` :
  * titre du graphique en paragraphe, s'il existe ;
  * données en tableau : catégories en lignes, séries en colonnes, noms de séries et valeurs tels que stockés dans le graphique (valeurs en cache), format d'affichage appliqué s'il est défini ; cellule d'angle vide si l'axe des catégories n'a pas de titre.
* Extraction impossible : `<!-- MISSING: graphique natif « titre », données non extraites -->`.

## Objets non extractibles

* Objet incorporé (OLE, pièce jointe, vidéo, audio), zone sans couche texte, glyphes non décodables (`(cid:12)`, police sans table Unicode) : `<!-- MISSING: nature, emplacement -->` à l'endroit concerné.
* Un marqueur `MISSING` décrit la nature et l'emplacement de ce qui manque, **jamais son contenu supposé**.

# Marqueurs réservés

Les seuls ajouts autorisés dans un .md sont les commentaires HTML suivants :

| Marqueur                                                      | Emplacement et usage                                                                      |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `<!-- SOURCE: nom.ext -->`                                    | Première ligne (après le front matter éventuel d'un fichier Markdown source)              |
| `<!-- HTML-TITLE: texte -->`                                  | Élément `<title>` d'une page HTML, juste après `SOURCE`                                   |
| `<!-- PAGE: N -->`                                            | Début de chaque page PDF, N = numéro physique à partir de 1                               |
| `<!-- SLIDE: N -->`                                           | Début de chaque diapositive                                                               |
| `<!-- SHEET: nom -->`                                         | Début de chaque feuille de tableur                                                        |
| `<!-- PAGE-HEADER:START -->` … `<!-- PAGE-HEADER:END -->`     | En-têtes, filigranes et logos d'en-tête, une fois, en début de fichier                    |
| `<!-- PAGE-FOOTER:START -->` … `<!-- PAGE-FOOTER:END -->`     | Pieds de page, une fois, en fin de corps                                                  |
| `<!-- HIDDEN:START -->` … `<!-- HIDDEN:END -->`               | Contenu masqué, à son emplacement                                                         |
| `<!-- HIDDEN-ROW -->`                                         | Ligne masquée d'un tableau HTML, juste avant son `<tr>`                                   |
| `<!-- HIDDEN-COLUMNS: C, F -->`                               | Colonnes masquées d'un tableau, juste avant le tableau                                    |
| `<!-- COMMENT (Auteur, AAAA-MM-JJ): texte -->`                | Commentaire de relecture, juste après le passage commenté                                 |
| `<!-- COMMENT (Auteur, AAAA-MM-JJ) [B4]: texte -->`           | Commentaire ou note de cellule de tableur, après le tableau, avec la référence de cellule |
| `<!-- COMMENT-REPLY (Auteur, AAAA-MM-JJ): texte -->`          | Réponse à un commentaire, juste après le commentaire parent                               |
| `<!-- SPEAKER-NOTES:START -->` … `<!-- SPEAKER-NOTES:END -->` | Notes de l'orateur, en fin de diapositive                                                 |
| `<!-- CHART:START -->` … `<!-- CHART:END -->`                 | Données d'un graphique natif                                                              |
| `<!-- MISSING: description -->`                               | Contenu non extractible                                                                   |
| `<!-- SOURCE-COMMENT: texte -->`                              | Commentaire présent dans une source HTML ou XML                                           |
| `<!-- XML-DOCTYPE: texte -->`, `<!-- XML-PI: texte -->`       | Déclaration DOCTYPE et instructions de traitement d'une source XML                        |

Règles :

* Le contenu situé **entre** deux marqueurs de début et de fin (en-têtes, contenu masqué, notes de l'orateur, données de graphique) est du texte Markdown normal, visible au rendu.
* La charge utile d'un marqueur issue de la source (texte d'un commentaire, auteur, nom de feuille) est du contenu : reproduite à l'identique. Une charge utile contenant `-->` est écrite avec `--&gt;` et signalée dans le rapport.
* Auteur ou date absent dans la source : élément omis ; parenthèses omises si les deux manquent.
* Un marqueur placé seul occupe sa propre ligne, entouré de lignes vides. Un marqueur placé au milieu d'un paragraphe (changement de page PDF, passage commenté, texte masqué en ligne) reste dans la ligne et ne commence jamais une ligne : `fin de la<!-- PAGE: 5 --> phrase`.
* Aucun autre commentaire HTML n'est ajouté.

# Règles par format

## DOCX (et DOC, RTF, ODT)

* DOC, RTF, ODT : conteneur converti en DOCX avant traitement.
* Révisions suivies : **version finale** (insertions acceptées, suppressions retirées, modifications de mise en forme ignorées).
* Numérotation automatique matérialisée (listes, titres, légendes).
* Champs : résultat affiché enregistré dans le fichier (dates, renvois, légendes) ; champs de numéro de page des en-têtes et pieds supprimés.
* Table des matières : conservée telle quelle (entrées et numéros de page ; points de conduite s'ils sont des caractères, tabulation de conduite rendue par une tabulation brute), sans lien.
* Zones de texte, formes, SmartArt, WordArt : texte inséré à son emplacement logique.
* En-têtes, pieds de page et filigranes : transformation n° 2.
* Notes de bas et de fin de page : `[^n]`.
* Commentaires et réponses : marqueurs `COMMENT` et `COMMENT-REPLY`.
* Texte masqué : marqueurs `HIDDEN`.
* Équations : LaTeX.
* Propriétés du document (auteur, titre, mots-clés) : non reproduites.

## PDF

* La couche texte de chaque page est vérifiée.
* Page ou zone sans couche texte (scan, texte présent uniquement dans une image) : **aucune OCR**. L'image est extraite et liée comme toute image, suivie de `<!-- MISSING: page N sans couche texte, texte non extrait -->`.
* Couche texte invisible préexistante (PDF déjà océrisé en amont) : extraite comme texte normal, signalée dans le rapport (exactitude non garantie par la source).
* Ordre de lecture logique : colonnes lues l'une après l'autre ; encadrés et notes latérales placés à la fin du bloc qu'ils accompagnent, jamais mêlés au flux.
* Reflow, césures, en-têtes et pieds répétés, numéros de page : transformations n° 1 et 2.
* Marqueur `PAGE` au début de chaque page, en ligne si la page change au milieu d'un paragraphe.
* Titres : taille et graisse de police, signets (le signet sert à identifier un titre, il n'est pas reproduit).
* Tableaux : détectés et reconstruits ; structure incertaine : tableau HTML fidèle ou, à défaut, texte des cellules dans l'ordre de lecture, signalé dans le rapport.
* Notes de bas de page : converties en `[^n]` seulement si l'appel et le texte de la note sont identifiables sans doute ; sinon, texte laissé à sa place comme texte courant.
* Annotations porteuses de texte (notes, commentaires de relecture) : marqueurs `COMMENT`. Surlignages sans texte : ignorés.
* Champs de formulaire : valeur saisie écrite à l'emplacement du champ ; case à cocher rendue par ☑ ou ☐ ; champ vide : rien.
* Glyphes non décodables : marqueur `MISSING`, signalé dans le rapport.
* Fichiers joints au PDF : marqueur `MISSING` (nature, nom).

## PPTX (et PPT, ODP)

* PPT, ODP : conteneur converti en PPTX avant traitement.
* Chaque diapositive, dans l'ordre : `<!-- SLIDE: N -->`, titre de la diapositive en titre Markdown (même niveau pour toutes les diapositives), puis contenu dans l'ordre de lecture (espaces réservés, puis formes de haut en bas et de gauche à droite, groupes développés).
* Diapositive sans titre : aucun titre inventé.
* Tableaux, images, graphiques, équations : conventions communes.
* Texte des masques et dispositions (pied de page, mention fixe) : transformation n° 2, une seule fois ; numéros de diapositive supprimés.
* Notes de l'orateur : entre `SPEAKER-NOTES:START` et `SPEAKER-NOTES:END`, en fin de diapositive.
* Commentaires : marqueurs `COMMENT`.
* Diapositive masquée : entièrement entre `HIDDEN:START` et `HIDDEN:END`, marqueur `SLIDE` compris.

## XLSX (et XLS, ODS)

* XLS, ODS : conteneur converti en XLSX avant traitement.
* Chaque feuille, dans l'ordre du classeur : `<!-- SHEET: nom -->`, puis titre Markdown portant le nom de la feuille (le nom d'onglet est du contenu visible).
* Valeur de cellule : **valeur affichée**, telle qu'Excel l'affiche (format de nombre, date, pourcentage, devise appliqué). Formules non écrites. Valeur en cache absente : recalcul par LibreOffice headless ; recalcul impossible : `<!-- MISSING: valeur de B4 non calculée -->` dans la cellule.
* Plage utilisée uniquement : lignes et colonnes vides en bordure ignorées. Une ligne ou colonne entièrement vide peut séparer deux tableaux distincts si la structure le justifie ; sinon, elle est conservée.
* Ligne d'en-tête : première ligne si elle joue ce rôle ; sinon, tableau HTML sans `<thead>`.
* Cellules fusionnées, lignes ou colonnes masquées : tableau HTML, marqueurs `HIDDEN-ROW` et `HIDDEN-COLUMNS`.
* Feuille masquée (y compris « très masquée ») : entièrement entre `HIDDEN:START` et `HIDDEN:END`, marqueur `SHEET` compris.
* Commentaires et notes de cellule : marqueurs `COMMENT` avec référence de cellule, après le tableau.
* Graphiques natifs et images : après les tableaux de la feuille, dans l'ordre de leur ancrage.
* Mise en forme conditionnelle, validation de données, couleurs : ignorées.

## CSV et TSV

* Encodage détecté ; délimiteur détecté (virgule, point-virgule, tabulation, barre verticale) ; guillemets d'encadrement interprétés selon la RFC 4180.
* Valeurs écrites comme chaînes brutes : aucun reformatage de nombre, de date, de zéros de tête.
* Tableau GFM, première ligne en en-tête ; barre verticale échappée `\|` ; retour à la ligne dans une cellule : `<br>` ; cellule vide : vide.
* Repli en bloc ` ```csv ` (ou ` ```tsv `) verbatim, pour tout le fichier (texte décodé exact, délimiteurs et guillemets compris), si : aucune ligne d'en-tête identifiable, nombre de colonnes incohérent entre les lignes, ou guillemets non analysables. Aucun seuil de largeur ni de taille. Repli signalé dans le rapport.

## TXT et autres textes bruts

* Encodage détecté.
* Structure déduite librement (titres, listes, paragraphes, tableaux) dans le cadre des transformations autorisées : reflow si la largeur fixe est manifeste, lignes décoratives, puces, blancs.
* Contenu de nature machine (journal, sortie de commande, configuration, code) : bloc de code verbatim, étiquette de langage seulement si évidente.

## JSON

* Analyse conservant l'ordre des clés, les clés dupliquées et la forme lexicale exacte des nombres (`1.0` reste `1.0`, `1e3` reste `1e3`) ; échappements de chaîne décodés.
* Structure Markdown interprétée, **sans marquage de type et sans repli**, quelle que soit la profondeur :
  * objets des premiers niveaux en titres tant que la hiérarchie le permet, niveaux suivants en listes imbriquées sans limite de profondeur ;
  * paire clé et valeur scalaire : `clé: valeur` (les deux-points transposent ceux du JSON) ;
  * tableau de scalaires : liste ;
  * tableau d'objets partageant les mêmes clés : tableau Markdown, colonnes dans l'ordre de première apparition, clé absente : cellule vide ;
  * tableau hétérogène : liste dont chaque élément est développé selon sa nature ;
  * `null`, `true`, `false` et nombres : écrits littéralement, comme du texte ;
  * chaîne vide, objet vide, tableau vide : clé suivie de rien ;
  * chaîne multiligne : retours forcés (`<br>` dans un tableau) ;
  * clé dupliquée : chaque occurrence écrite, dans l'ordre.
* Ambiguïtés acceptées : `"42"` et `42`, `"null"` et `null`, chaîne vide et valeur vide deviennent indiscernables.

## YAML

* Bloc ` ```yaml ` verbatim : texte décodé exact, commentaires, ancres, documents multiples et indentation compris.

## XML

* Analyse conservant commentaires et sections CDATA ; entités résolues.
* Déclaration XML (`<?xml …?>`) : non reproduite. DOCTYPE et instructions de traitement : marqueurs `XML-DOCTYPE` et `XML-PI` à leur position.
* Structure Markdown interprétée, sans repli :
  * élément : titre tant que la hiérarchie le permet, sinon élément de liste imbriquée, libellé du nom qualifié tel qu'écrit (`ns:element`) ;
  * attributs : `@nom: valeur` sous l'élément, dans l'ordre de la source, déclarations `xmlns` comprises ;
  * élément à contenu texte seul : `nom: texte` ;
  * contenu mixte (`<p>Texte <b>gras</b> suite</p>`) : texte continu dans l'ordre, texte des éléments enfants en ligne ; les attributs éventuels de ces enfants sont listés en sous-éléments, rien n'est perdu ;
  * éléments frères répétés de même structure : tableau possible, colonnes `@attributs` et éléments enfants ;
  * commentaires XML : `<!-- SOURCE-COMMENT: … -->` à leur position ;
  * CDATA : texte ;
  * nœuds texte composés uniquement de blancs d'indentation : ignorés.
* XHTML : règles HTML.

## HTML

* Conversion en Markdown du contenu visible du `<body>`.
* `<title>` : marqueur `HTML-TITLE`.
* Non reproduits (non visibles) : reste de `<head>`, `<script>`, `<style>`, `<template>`, `<noscript>`, attributs de style et de classe.
* Éléments masqués (`hidden`, `display:none` en style en ligne) : marqueurs `HIDDEN`.
* Navigation, menus, pieds de site : conservés.
* Commentaires HTML de la source : marqueurs `SOURCE-COMMENT`.
* `<u>`, `<sup>`, `<sub>` conservés ; `<mark>`, `<span>`, `<font>` : texte seul.
* Images locales : copiées dans le dossier d'images ; images distantes : URL conservée.

## Markdown

* Recopié tel quel. Seuls changements : encodage UTF-8, fins de ligne LF, ajout du marqueur `SOURCE` (après le front matter éventuel). Liens d'images inchangés.

## Autres formats

* Identifier la nature du fichier (document, tableur, présentation, données, texte, image) et appliquer les règles du format le plus proche, au besoin après conversion du conteneur sans perte de texte (LibreOffice, pandoc ; EPUB traité comme HTML, chapitre par chapitre).
* Format textuel non listé (INI, TOML, SQL, code source, etc.) : bloc de code verbatim avec étiquette de langage.
* Image isolée (PNG, JPG, SVG, etc.) : copiée dans le dossier d'images, référencée `![](chemin)`, suivie de `<!-- MISSING: texte éventuel de l'image non extrait (pas d'OCR) -->`.
* Archive, audio, vidéo, binaire sans texte : aucun .md, signalé dans le rapport.
* Fichier chiffré, protégé ou corrompu : aucun .md, échec signalé dans le rapport. Jamais de .md partiel silencieux.

# Contrôle de fidélité

Avec exécution de code, pour chaque fichier, avant livraison :

1. Extraire un **texte de référence** de la source, de préférence avec un outil différent de celui de la conversion (par exemple `pdftotext` contre PyMuPDF, XML brut contre pandoc).
2. Produire le **texte rendu** du .md : rendu GFM puis texte brut, commentaires HTML exclus.
3. Appliquer aux deux textes les seules transformations autorisées, puis normaliser les blancs.
4. Comparer les séquences de mots. Toute différence non expliquée par une transformation autorisée est corrigée dans le .md, puis le contrôle est relancé.
5. Formats de données (CSV, JSON, XML) : comparer la suite ordonnée des clés, noms et valeurs de la source à celle du .md.
6. Contrôler séparément la charge utile des marqueurs (commentaires, auteurs, noms de feuille) contre la source.
7. Vérifier le rendu : tableaux bien formés, aucune emphase, liste ou titre involontaire, aucun marqueur en début de ligne suivi de texte, liens d'images résolus.

Sans exécution de code : relecture section par section contre la source, en vérifiant en priorité chiffres, noms propres, URL, ponctuation et caractères spéciaux.

# Rapport de conversion

Le rapport est livré **dans la réponse**, jamais dans les fichiers .md. Pour chaque fichier source :

* fichier produit et dossier d'images éventuel ;
* statut : complet, partiel (marqueurs `MISSING` présents) ou échec, avec la cause ;
* mode d'extraction : programmatique ou lecture directe ;
* replis appliqués (CSV en bloc, tableaux HTML, tableaux PDF incertains) ;
* contenus non extraits (liste des marqueurs `MISSING`) ;
* points d'attention : césures ambiguës conservées, couche OCR préexistante, `-->` neutralisés, formules PDF non reconstruites ;
* résultat du contrôle de fidélité : identique, ou écarts résiduels listés avec leur position.

# Résultat attendu

Produire un fichier `.md` par fichier source, fidèle à son contenu, accompagné du rapport de conversion.

Le succès de la tâche est évalué selon cet ordre de priorité :

1. préservation intégrale et exacte du texte ;
2. exactitude structurelle (titres, listes, tableaux, notes, blocs de code) ;
3. application des conventions (mise en forme, marqueurs, nommage).

Un élément de structure approximatif est un défaut mineur. Un caractère ajouté, supprimé ou modifié hors transformation autorisée est un échec.