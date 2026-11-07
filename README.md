# Cas Zéro – Simulateur de cas cliniques (PWA)

Application statique (HTML/CSS/JS pur, sans build) qui fait jouer un cas clinique à un étudiant en médecine.

## Structure

```
index.html, styles.css, app.js      application (logique en anglais)
data/interface.json                 textes communs à tous les cas (boutons, messages)
data/cas-index.json                 liste des cas disponibles
data/cas-*.json                     un fichier par cas clinique (contenu en français)
modeles/modele-cas.json             modèle vide à copier pour un nouveau cas
tools/validate-case.js              contrôle qu'un cas est complet et cohérent
tools/test-engine.js                tests automatiques du moteur
manifest.webmanifest, sw.js, icons/ couche PWA (installable, hors ligne)
GUIDE-NOUVEAU-CAS.md                comment écrire un nouveau cas
```

## Tester en local

```bash
npx serve .
```

Ouvrez l'adresse affichée. L'application doit passer par un serveur (les fichiers JSON sont chargés avec `fetch`).

## Contrôles automatiques (Node requis)

```bash
node tools/validate-case.js     # valide tous les cas de data/cas-index.json
node tools/test-engine.js       # tests du moteur sur le premier cas
```

## Déployer (GitHub + Vercel)

Envoyez tous les fichiers à la racine d'un dépôt GitHub, puis importez le dépôt dans Vercel (framework « Other », aucun build). Chaque modification sur GitHub redéploie le site automatiquement.

Après un changement, incrémentez `CACHE_NAME` dans `sw.js` pour que les téléphones prennent la nouvelle version.

## Règles de score

- Score de départ 100, minimum 0, **uniquement des pénalités**
- « Besoin d'aide ? » : -2 (une fois par étape)
- Examen neurologique oublié : -10
- Examen inutile : -10
- Examen sensible sans précaution (PL sans imagerie ni fond d'œil) : -10
- Diagnostic non trouvé (réponse révélée) : -10 chacun
- Erreur fatale (sédatifs, ignorer la PL) : -30

Les constantes non précisées par un cas sont complétées par des valeurs normales (`data/interface.json`). Toutes les valeurs de score sont dans le JSON du cas. Voir `GUIDE-NOUVEAU-CAS.md`.
