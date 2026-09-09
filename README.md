# Programme d'entraînement — Krakens Dodgeball Club

Application statique (une seule page HTML, aucun serveur, aucune dépendance) pour préparer les entraînements du club :

- **Exercices** : créer, modifier, dupliquer et supprimer des fiches d'exercice (but, objectifs, consignes, durée), les classer par catégorie (Échauffement, Technique, Renforcement, Tactique, Jeu), les retrouver par recherche texte ou par filtre de catégorie, les lier à un schéma tactique, les exporter/importer en JSON. C'est une vraie bibliothèque réutilisable d'une séance à l'autre, pas une liste jetable.
- **Séances** : construire la séance du jour en ajoutant des exercices choisis dans la bibliothèque, dans l'ordre voulu, avec le total de durée comparé à une durée cible en temps réel — pas un simple instantané de toute la bibliothèque.
- **Schémas tactiques** : dessiner sur le terrain de dodgeball officiel avec joueurs, plots, flèches, zones, textes et notes.
- **Export PDF** : chaque séance sauvegardée peut être exportée en PDF (via l'impression native du navigateur) avec sommaire cliquable, prêt à imprimer ou partager.

## Utilisation

Aucune installation : ouvrez `index.html` dans un navigateur, ou servez le dossier avec n'importe quel serveur statique (GitHub Pages, par exemple).

**Les données (exercices, schémas, séances) sont synchronisées entre coachs** via un webhook n8n auto-hébergé (voir section Synchronisation ci-dessous). Le `localStorage` du navigateur reste utilisé comme cache local/hors-ligne — l'app fonctionne même sans connexion, et se resynchronise dès que possible.

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

## Synchronisation multi-coachs

Les exercices, schémas et séances sont partagés entre tous les coachs via un workflow n8n (**« KDC - Sync Entrainements »**, auto-hébergé sur `n8n.laboiteaoutia.fr`) qui expose 9 routes webhook (GET/POST/DELETE × exercices/schémas/séances) adossées à 3 [Data Tables](https://docs.n8n.io/data-tables/) n8n (`kdc_exercises`, `kdc_schemas`, `kdc_sessions`).

**Fonctionnement :**
- Au chargement de la page (et via le bouton **« 🔄 Actualiser »**), l'app récupère les données du serveur et les affiche — le serveur est la source de vérité.
- Chaque création/suppression (exercice, schéma, séance) est aussitôt poussée au serveur en arrière-plan, sans bloquer l'action locale.
- Si le réseau est indisponible, l'app continue de fonctionner avec les dernières données connues en `localStorage` (indicateur "⚠️ Hors ligne" affiché) ; les modifications faites hors-ligne restent locales tant qu'aucune synchro n'a réussi.
- **Pas de fusion fine en cas de modifications concurrentes** : un "Actualiser" remplace entièrement les données locales par celles du serveur. Pour un club à quelques coachs avec peu d'éditions simultanées, ce n'est pas un problème en pratique ; ça le deviendrait avec un usage plus intensif.

**Accès :** chaque coach doit renseigner une fois la **clé de synchro** (bouton **« 🔑 Clé de synchro »**, mémorisée ensuite dans son navigateur) — demandez-la à Quentin. Sans clé, l'app fonctionne uniquement en local (comme avant la synchro).

**Sécurité :** la clé est vérifiée par n8n sur chaque appel (`options.onlyRunIf` sur le nœud Webhook) — une requête avec une clé absente ou incorrecte reçoit une réponse `200` vide sans qu'aucune donnée ne soit lue ni écrite. La clé elle-même est en clair dans le workflow n8n (visible uniquement par vous en tant que propriétaire de l'instance) — suffisant pour ce contexte (club amateur, données non sensibles), mais une vraie authentification par credential n8n dédié (chiffré) serait l'étape suivante si besoin d'un niveau de sécurité supérieur.

## Corrections apportées à la version initiale

- **Faille XSS** : le nom, les consignes, les notes, les plots et les schémas provenaient directement du `localStorage` ou d'un fichier JSON importé et étaient injectés tels quels via `innerHTML`. Un exercice importé avec un nom du type `<img src=x onerror=...>` aurait exécuté du code arbitraire dans le navigateur du coach. Tout texte utilisateur est désormais échappé (`escapeHtml`) avant affichage.
- **Modal non scrollable** : sur un écran de faible hauteur, le formulaire de création d'exercice dépassait la fenêtre sans qu'aucun défilement ne soit possible (le `overflow-y: auto` n'existait que dans la règle mobile). Corrigé pour toutes les tailles d'écran.
- **Quota `localStorage` dépassé** : si le stockage local est plein (beaucoup de schémas avec vignettes), les sauvegardes échouaient silencieusement. Un message d'erreur explicite s'affiche maintenant et l'action est annulée proprement.

## Améliorations ajoutées après la première version

- **Annuler / Rétablir sur le dessin de schéma** : chaque élément ajouté ou supprimé sur le terrain (joueur, flèche, zone, plot...) pousse un état dans un historique. Boutons dédiés « ↩️ Annuler » / « ↪️ Rétablir » dans le panneau Actions, ou raccourcis clavier `Ctrl+Z` / `Ctrl+Y` (`Ctrl+Shift+Z` fonctionne aussi pour rétablir). L'historique repart de zéro à chaque chargement d'un schéma sauvegardé.
- **Accessibilité clavier** : les onglets « Exercices » / « Créer des Schémas » sont désormais de vrais `<button role="tab">` navigables au clavier (avec `aria-selected` à jour), les plots de la liste sont sélectionnables au clavier (`Tab` puis `Entrée`/`Espace`), tous les boutons icône-seule (suppression d'exercice, de plot) ont un `aria-label` explicite, et un contour de focus visible (`:focus-visible`) apparaît sur tous les éléments interactifs pour la navigation au clavier.
- **Gestion proactive du stockage local** : un indicateur (« Stockage local : X Ko (~Y%) ») s'affiche en temps réel dans l'onglet Schémas, avec un code couleur (vert / orange dès 70% / rouge dès 90%) — pour voir venir la limite avant qu'elle ne bloque une sauvegarde.
- **Bibliothèque d'exercices par catégorie** : chaque exercice a désormais une catégorie obligatoire (🔥 Échauffement, 🎯 Technique, 💪 Renforcement, 🧠 Tactique, 🎮 Jeu). Des filtres au-dessus de la grille (avec compteur par catégorie) permettent de retrouver rapidement les exercices d'un type donné plutôt que de faire défiler une liste plate. Les exercices importés d'avant cet ajout tombent automatiquement dans « Non classé ».
- **Panier de séance (correction du principal manque fonctionnel)** : jusque-là, « Sauvegarder la séance » snapshottait *toute* la bibliothèque d'exercices sans distinction — avec 40 exercices en fin de saison, chaque séance sauvegardée aurait contenu les 40. Un panier « 🧺 Séance en cours » permet maintenant de choisir précisément les exercices du jour (bouton « ➕ Ajouter à la séance » sur chaque carte), de les réordonner (▲▼), avec un total de durée en direct comparé à une durée cible réglable (vert si proche, orange/rouge si trop loin). Seuls les schémas réellement liés aux exercices choisis sont embarqués dans la séance sauvegardée. Le panier persiste si on recharge la page en pleine préparation, et se vide automatiquement après sauvegarde.
- **Barre d'actions allégée** : sur mobile, les 6 boutons pleine largeur de l'en-tête (import/export/synchro) forçaient à défiler avant de voir le premier exercice. Les actions secondaires sont maintenant regroupées derrière un menu « ⋯ Plus d'actions », ne laissant en avant que « Créer un exercice ».
- **Modifier un exercice existant** : chaque carte a maintenant un bouton « ✏️ Modifier » qui rouvre le formulaire pré-rempli et met à jour l'exercice en place (même identifiant, donc les séances qui le référencent restent cohérentes), au lieu d'obliger à supprimer et tout retaper pour la moindre correction.
- **Dupliquer un exercice** : bouton « ⧉ Dupliquer » sur chaque carte pour créer une variante d'un exercice existant (nom suffixé « (copie) ») sans repartir de zéro.
- **Recherche texte sur les exercices** : un champ de recherche au-dessus des filtres de catégorie retrouve un exercice par son nom, cumulable avec le filtre de catégorie actif — utile dès que la bibliothèque dépasse la vingtaine d'exercices.
- **Sélecteur de terrain multi-sports retiré** de l'onglet Schémas (volleyball/basketball/football/handball/badminton/rugby) — hérité du code initial, sans utilité réelle pour un club de dodgeball. Le terrain dessiné est désormais toujours le terrain de dodgeball officiel, sans option superflue.
- **Nom de plot sans interruption** : poser un plot ne demande plus son nom à chaque fois (nom par défaut « Plot N » automatique) — le nom se change ensuite d'un double-clic sur le plot dans la liste « Plots placés », qui a aussi été allégée visuellement (elle fait doublon avec les noms déjà affichés sur le terrain ; elle ne sert plus qu'à renommer/supprimer un plot précis sans avoir à viser sur le dessin).
- **Miniatures de schéma en PNG pleine résolution** : les images de schéma étaient compressées en JPEG à taille réduite pour économiser du stockage, ce qui les rendait illisibles une fois agrandies dans l'export PDF. Un schéma dessiné (aplats de couleur, texte) compresse presque aussi bien en PNG sans perte (~35 Ko, comparable au JPEG compressé) — retour à la pleine résolution sans le compromis de netteté.
- **Export en vrai PDF plutôt qu'en faux `.doc`** : l'export de séance ouvrait un fichier `.doc` (en réalité du HTML) que Word interprétait mal — mise en page cassée entre les pages, image du schéma étirée et floue. Le bouton « 📄 PDF » déclenche maintenant l'impression native du navigateur (l'utilisateur choisit « Enregistrer en PDF » comme imprimante) : rendu net, pagination correctement respectée, sans dépendance externe.

## Limites connues / axes d'amélioration

- **Pas de fusion fine en cas de modifications concurrentes** (voir section Synchronisation ci-dessus) — dernier "Actualiser" gagne.
- **`localStorage` reste plafonné (~5-10 Mo)** — chaque schéma pèse environ 35 Ko en PNG plein format. `localStorage` sert maintenant de cache (le serveur est la source de vérité), la limite est donc moins critique qu'avant la synchro, mais reste un point de vigilance si le club accumule plusieurs centaines de schémas.
- **Clé de synchro en clair dans le workflow n8n** (voir section Synchronisation ci-dessus) — acceptable pour ce contexte, mais pas une vraie authentification chiffrée.
- **Pas de modèle de séance réutilisable** (ex. "Séance standard : 10 min échauffement + 20 min technique + 20 min jeu") — chaque séance se reconstruit dans le panier à partir de zéro.
- **Pas de minuteur intégré** pour chronométrer un exercice pendant la séance elle-même.
- **Logo à intégrer manuellement** (voir section Logo ci-dessus).
