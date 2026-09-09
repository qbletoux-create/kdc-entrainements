# Programme d'entraînement — Krakens Dodgeball Club

Application statique (une seule page HTML, aucun serveur, aucune dépendance) pour préparer les entraînements du club :

- **Exercices** : créer des fiches d'exercice (but, objectifs, consignes, durée), les lier à un schéma tactique, les exporter/importer en JSON, et assembler une séance complète.
- **Schémas tactiques** : dessiner sur un terrain (dodgeball par défaut, ou volleyball/basketball/football/handball/badminton/rugby pour varier les entraînements) avec joueurs, plots, flèches, zones, textes et notes.
- **Export Word** : chaque séance sauvegardée peut être exportée en `.doc` (HTML) avec sommaire cliquable, prêt à imprimer ou partager.

## Utilisation

Aucune installation : ouvrez `index.html` dans un navigateur, ou servez le dossier avec n'importe quel serveur statique (GitHub Pages, par exemple).

**Important : toutes les données (exercices, schémas, séances) sont stockées dans le `localStorage` du navigateur.** Rien n'est envoyé sur un serveur, mais rien n'est partagé entre coachs/appareils non plus — voir la section Limites ci-dessous. Pensez à utiliser régulièrement le bouton **« Exporter les exercices »** pour garder une sauvegarde du travail.

## Charte graphique

L'interface reprend la charte graphique officielle du club (2026), synchronisée avec le site [kdc-web](https://github.com/rharkor/kdc-web) :

| Couleur | Usage | Hex |
|---|---|---|
| Abysse | Fond du header, textes foncés | `#243455` |
| Marine | Bleu de marque, actions principales | `#015394` |
| Écume | Surfaces claires, bordures | `#d2e1ec` |
| Ambre | Accent chaud, CTA secondaires | `#f8bf4e` |
| Soleil | Accent rare | `#fdc503` |
| Nacre | Blanc | `#ffffff` |

Polices : **Montserrat** (titres, en majuscules) et **Karla** (texte courant), chargées depuis Google Fonts.

### Logo

Le logo officiel du club (`public/brand/logo-256.png` dans le repo `kdc-web`, repo privé) n'a pas pu être récupéré automatiquement lors de la création de ce dépôt — l'assistant IA qui a préparé ce repo n'avait accès qu'à un aperçu visuel du fichier, pas à ses données binaires. Le header utilise pour l'instant un badge textuel « 17 ». Pour ajouter le vrai logo :

1. Téléchargez `public/brand/logo-256.png` (ou `logo.png`) depuis le repo `kdc-web`.
2. Déposez-le dans ce dépôt sous `assets/logo.png`.
3. Dans `index.html`, remplacez le `<div class="header-badge">17</div>` par `<img src="assets/logo.png" alt="Krakens Dodgeball Club" class="header-badge" style="border-radius:0;background:none;">` (ajustez la classe/taille si besoin).

## Corrections apportées à la version initiale

- **Faille XSS** : le nom, les consignes, les notes, les plots et les schémas provenaient directement du `localStorage` ou d'un fichier JSON importé et étaient injectés tels quels via `innerHTML`. Un exercice importé avec un nom du type `<img src=x onerror=...>` aurait exécuté du code arbitraire dans le navigateur du coach. Tout texte utilisateur est désormais échappé (`escapeHtml`) avant affichage.
- **Modal non scrollable** : sur un écran de faible hauteur, le formulaire de création d'exercice dépassait la fenêtre sans qu'aucun défilement ne soit possible (le `overflow-y: auto` n'existait que dans la règle mobile). Corrigé pour toutes les tailles d'écran.
- **Quota `localStorage` dépassé** : si le stockage local est plein (beaucoup de schémas avec vignettes), les sauvegardes échouaient silencieusement. Un message d'erreur explicite s'affiche maintenant et l'action est annulée proprement.

## Améliorations ajoutées après la première version

- **Annuler / Rétablir sur le dessin de schéma** : chaque élément ajouté ou supprimé sur le terrain (joueur, flèche, zone, plot...) pousse un état dans un historique. Boutons dédiés « ↩️ Annuler » / « ↪️ Rétablir » dans le panneau Actions, ou raccourcis clavier `Ctrl+Z` / `Ctrl+Y` (`Ctrl+Shift+Z` fonctionne aussi pour rétablir). L'historique repart de zéro à chaque chargement d'un schéma sauvegardé.
- **Accessibilité clavier** : les onglets « Exercices » / « Créer des Schémas » sont désormais de vrais `<button role="tab">` navigables au clavier (avec `aria-selected` à jour), les plots de la liste sont sélectionnables au clavier (`Tab` puis `Entrée`/`Espace`), tous les boutons icône-seule (suppression d'exercice, de plot) ont un `aria-label` explicite, et un contour de focus visible (`:focus-visible`) apparaît sur tous les éléments interactifs pour la navigation au clavier.
- **Gestion proactive du stockage local** : un indicateur (« Stockage local : X Ko (~Y%) ») s'affiche en temps réel dans l'onglet Schémas, avec un code couleur (vert / orange dès 70% / rouge dès 90%) — pour voir venir la limite avant qu'elle ne bloque une sauvegarde. Les miniatures de schéma sont désormais générées en JPEG compressé à taille réduite (450×300, qualité 70%) plutôt qu'en PNG plein format (900×600), ce qui réduit fortement l'empreinte de chaque schéma sauvegardé.

## Limites connues / axes d'amélioration

- **Pas de synchronisation entre appareils.** Chaque navigateur a ses propres données. Pour une utilisation par plusieurs coachs, il faudrait soit un petit backend (ex. un Airtable ou une base de données via n8n) pour centraliser exercices et séances, soit au minimum une routine "exporter/importer" partagée. C'est le chantier le plus structurant restant — un choix d'architecture à faire avant de s'y attaquer.
- **`localStorage` reste plafonné (~5-10 Mo)** malgré la compression des miniatures. Au-delà de plusieurs dizaines de schémas, le stockage peut encore se remplir — l'indicateur de stockage prévient avant que ça arrive, mais une migration vers IndexedDB (limite bien plus haute) resterait la solution définitive.
- **Pas d'authentification.** N'importe qui avec le lien (si hébergé publiquement, ex. GitHub Pages) peut utiliser l'outil, mais chacun a ses propres données locales — il n'y a pas de fuite d'un coach à l'autre, juste pas de partage non plus.
- **Logo à intégrer manuellement** (voir section Logo ci-dessus).
