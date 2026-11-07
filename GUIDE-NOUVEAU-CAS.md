# Guide : écrire un nouveau cas clinique

Le cas « méningite / paludisme » sert de modèle. Un nouveau cas ne demande **aucun code** : on remplit un fichier JSON.

## 1. Les 5 étapes

1. Copiez `modeles/modele-cas.json` vers `data/cas-NOM-DU-CAS.json`.
2. Remplissez tous les champs « À remplir » (voir les sections ci-dessous).
3. Vérifiez le fichier :
   ```bash
   node tools/validate-case.js data/cas-NOM-DU-CAS.json
   ```
   (ou, sans Node, relisez la liste de contrôle en fin de guide).
4. Ajoutez le cas dans `data/cas-index.json` :
   ```json
   { "id": "cas-nom-du-cas", "fichier": "data/cas-NOM-DU-CAS.json" }
   ```
   Le cas affiché par défaut est le premier de la liste. Pour ouvrir un autre cas : `https://votre-site.vercel.app/?cas=cas-nom-du-cas`.
5. Ajoutez le fichier dans la liste `APP_SHELL` de `sw.js` et changez `cas-zero-v4` en `cas-zero-v5` (pour le mode hors ligne), puis publiez sur GitHub.

## 2. Ce que contient un cas

| Section | Rôle |
|---|---|
| `metadonnees` | Difficulté et durée affichées sur l'écran d'accueil |
| `regles` | `penaliteAide` (points perdus par indice) et `tentativesDiagnosticAvantReponse` |
| `introduction` | Lieu, contexte, motif : écran « Aperçu » (les constantes viennent de l'étape 2) |
| `etape1_anamnese` | Informations révélées par les questions de l'étudiant |
| `etape2_examen_physique` | Constantes, examen général, examen des systèmes, examen clé à demander |
| `etape3_paraclinique` | Examens prioritaires, examens acceptables, malus d'examen inutile |
| `etape4_diagnostic` | Diagnostics attendus avec leurs points |
| `etape5_traitement` | Classes de traitement obligatoires, erreurs fatales |
| `etape6_debriefing` | Points à retenir (format « Titre : explication ») |

## 3. Règles de score

- On part de `scoreInitial` (100) et il n'y a **que des pénalités**. Le score ne passe jamais sous 0.
- Indice « Besoin d'aide ? » : `penaliteAide` (une fois par étape).
- Examen clé oublié à l'étape 2 : `malusSiOubli`.
- Examen sans rapport à l'étape 3 : `malusExamenInutile` (une seule fois par texte identique).
- Diagnostic que l'étudiant n'a pas trouvé (il clique sur « Voir la réponse ») : les `points` du diagnostic.
- Examen sensible demandé sans précaution préalable (ex. PL sans imagerie) : `malus` du `prerequis` (une seule fois par examen).
- Erreur fatale à l'étape 5 : `punition`.
- Total des pénalités fixes à garder sous le score initial (le validateur vous prévient).

## 4. Écrire de bons mots-clés

- Écrivez-les **sans accent, en minuscules**. Le moteur ignore de toute façon accents, majuscules et ponctuation.
- Un mot de **3 lettres ou moins** (`pl`, `ge`, `tdm`) doit apparaître comme mot entier : « pl » ne se déclenche pas dans « plasmodium ».
- Un mot de plus de 3 lettres est trouvé **à l'intérieur** d'un mot : `cepha` reconnaît « céphalées ».
- Les mots de 6 lettres ou plus tolèrent **une faute de frappe** (deux pour 10 lettres ou plus) : « ceftriaxon » est reconnu.
- Prévoyez les synonymes, les abréviations courantes et les noms commerciaux.
- Une négation juste avant un mot de l'étape 5 annule l'erreur fatale (« pas de midazolam »).

## 5. Anamnèse : une information par thème

Chaque `revelation` a ses propres `motsCles` et un `texte`. Une question de l'étudiant révèle toutes les informations dont les mots-clés correspondent. Une question vague (`questionsGenerales`, ex. « racontez-moi ») révèle la suivante dans l'ordre.

- `essentiel: true` : l'étudiant doit l'obtenir pour pouvoir continuer.
- `essentiel: false` : facultatif (signes négatifs, détails). S'il ne le demande pas, le débriefing le lui montre.

## 6. Adapter le cas au contexte (les « nuances »)

Ces champs changent l'esprit du cas sans toucher au moteur :

- **Lieu et ressources** : `introduction.lieu` et `introduction.contexte` (zone d'endémie, structure isolée, consultation, domicile...).
- **Constantes** : ne renseignez que celles qui sont citées ou pathologiques. Toutes les autres sont complétées automatiquement par des **valeurs normales** (`constantesParDefaut` dans `data/interface.json` : TA 120/80, FC 80, FR 16, SpO₂ 98 %, T° 37,0 °C, glycémie 100 mg/dl). Les six constantes sont toujours visibles dans l'aperçu, à l'examen physique et sur le moniteur. Une valeur du cas remplace la valeur normale.
- **Examens acceptables** : listez les examens raisonnables mais non prioritaires, avec `malus: 0` (ou un petit malus). Sans `resultat`, l'étudiant voit « aucun résultat disponible dans ce cas ».
- **Traitement** : une `groupesTraitement` par classe thérapeutique exigée. L'étudiant doit en couvrir **chacune**.
- **Difficulté** : jouez sur le nombre d'informations essentielles, les pièges (examen utile mais non prioritaire, erreur fatale plausible) et `tentativesDiagnosticAvantReponse`.

## 7. Examens avec prérequis (ex. imagerie avant la PL)

Certains examens exigent une précaution préalable. Dans `examensValides` :

- Les examens qui peuvent se **substituer** l'un à l'autre portent le même `groupeAlternatif` (ex. imagerie cérébrale **ou** fond d'œil). Un seul suffit pour valider le groupe.
- L'examen sensible porte un `prerequis` : `groupe` (le groupe à faire d'abord), `malus`, `libelleJournal` (intitulé dans le débriefing) et `message` (explication affichée à l'étudiant).
- Si l'étudiant demande l'examen sensible trop tôt : il est **bloqué** (pas de résultat), le malus s'applique une seule fois, puis l'examen passe dès qu'un examen du groupe est fait.
- Si les deux sont demandés dans le même message dans le bon ordre (« scanner puis PL »), aucune pénalité.
- Le bilan est complet quand tous les examens sans groupe sont faits **et** qu'un examen de chaque groupe est fait.

Voir `modeles/modele-cas.json` pour un exemple complet.

## 8. Liste de contrôle avant publication

- [ ] Constantes cohérentes avec le texte de l'examen (même fréquence cardiaque, etc.)
- [ ] Masculin/féminin cohérent partout (patient ou patiente)
- [ ] Chaque étape a un `aide` (sauf traitement et débriefing)
- [ ] Aucun mot-clé d'examen acceptable n'est aussi un examen prioritaire
- [ ] Les examens à risque (PL, etc.) ont leur `prerequis` si une précaution est nécessaire
- [ ] Aucun médicament attendu n'est aussi une erreur fatale
- [ ] Le cas a été joué en entier, une fois sans faute et une fois avec des erreurs
- [ ] Contenu médical relu et validé par un clinicien
