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

## Limites connues / axes d'amélioration

- **Pas de synchronisation entre appareils.** Chaque navigateur a ses propres données. Pour une utilisation par plusieurs coachs (vous notamment), il faudrait soit un petit backend (ex. un Airtable ou une base de données via n8n) pour centraliser exercices et séances, soit au minimum une routine "exporter/importer" partagée.
- **`localStorage` a une limite (~5-10 Mo).** Les vignettes de schémas (images PNG en base64) grossissent vite. Au-delà d'une trentaine de schémas, le stockage peut se remplir. Solution possible : compresser les vignettes, ou migrer vers IndexedDB (limite bien plus haute).
- **Pas d'authentification.** N'importe qui avec le lien (si hébergé publiquement, ex. GitHub Pages) peut utiliser l'outil et voir des données de démonstration, mais chacun a ses propres données locales — il n'y a pas de fuite d'un coach à l'autre, juste pas de partage non plus.
- **Accessibilité perfectible** : les boutons d'action rapide (✕ de suppression) n'ont pas tous un `aria-label`, et le contraste de certains textes gris sur fond clair mériterait une vérification WCAG.
- **Pas de undo/redo sur le dessin de schéma.** Une fois un élément placé sur le terrain, seule la suppression totale ("Effacer tout") ou la suppression d'un plot via la liste est possible — pas d'annulation ciblée d'une flèche ou d'une zone mal placée.
- **Logo à intégrer manuellement** (voir section Logo ci-dessus).
