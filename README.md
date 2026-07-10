# Suivi Qualité TERSEA — déploiement en ligne (GitHub Pages + Firebase)

Ce dossier contient un tableau de bord complet (Reporting, Suivi débriefing, Import,
Extractions) sous forme d'une seule page web (`index.html`). Une fois déployé, il est
accessible par une URL, partagé en temps réel entre toutes les personnes qui l'ouvrent :
un changement fait par l'un apparaît chez les autres presque instantanément.

Deux services gratuits sont nécessaires :
- **GitHub Pages** : héberge la page (l'affichage).
- **Firebase (Firestore)** : héberge les données (la base de données partagée).

Aucun des deux n'est payant pour ce niveau d'usage (équipe interne, quelques dizaines
d'utilisateurs, volumétrie de l'ordre de quelques centaines de lignes).

---

## Étape 1 — Créer le projet Firebase (~5 min)

1. Va sur [console.firebase.google.com](https://console.firebase.google.com) et connecte-toi
   avec un compte Google.
2. Clique **Ajouter un projet**, donne-lui un nom (ex. `tersea-qualite`), continue avec les
   options par défaut (Google Analytics n'est pas nécessaire, tu peux le désactiver).
3. Une fois le projet créé, dans le menu de gauche : **Build > Firestore Database**.
4. Clique **Créer une base de données**.
   - Choisis une région proche (ex. `eur3 (europe-west)`).
   - Choisis **Mode production** (les règles de sécurité seront collées à l'étape 3).
5. Toujours dans le menu de gauche : **Build > Authentication**.
   - Clique **Get started** / **Commencer**.
   - Dans l'onglet **Sign-in method**, active le fournisseur **Anonyme (Anonymous)**.
     (Cela permet à l'application de s'identifier techniquement sans demander de mot de
     passe à personne — c'est ce qui autorise la lecture/écriture selon les règles de
     l'étape 3.)

## Étape 2 — Récupérer la configuration de ton application web

1. Dans la Console Firebase : icône ⚙️ (Paramètres) en haut à gauche > **Paramètres du projet**.
2. Section **Vos applications**, clique l'icône `</>` (Web) pour ajouter une application web.
3. Donne-lui un nom (ex. `dashboard-qualite`), pas besoin de cocher Firebase Hosting.
4. Firebase affiche un bloc `firebaseConfig = { apiKey: "...", authDomain: "...", ... }`.
   **Copie-le.**
5. Ouvre le fichier `index.html` (dans un éditeur de texte simple, Bloc-notes suffit), cherche
   ce bloc vers le haut du fichier :

   ```js
   const firebaseConfig = {
     apiKey: "COLLE_TA_CLE_API_ICI",
     authDomain: "TON-PROJET.firebaseapp.com",
     projectId: "TON-PROJET",
     storageBucket: "TON-PROJET.appspot.com",
     messagingSenderId: "000000000000",
     appId: "1:000000000000:web:xxxxxxxxxxxxxxxxxx",
   };
   ```

   Remplace-le entièrement par le bloc copié à l'étape 4. Enregistre le fichier.

> **Ces clés ne sont pas secrètes.** Contrairement à un mot de passe, la clé API Firebase
> web est prévue pour être visible dans le code source d'une page — n'importe qui peut
> l'y voir avec "Afficher le code source". Ce qui protège réellement les données, ce sont
> les **règles Firestore** de l'étape 3, pas le fait de cacher cette configuration.

## Étape 3 — Appliquer les règles de sécurité Firestore

1. Dans la Console Firebase : **Build > Firestore Database > Règles** (onglet en haut).
2. Remplace tout le contenu par celui du fichier `firestore.rules` fourni ici.
3. Clique **Publier**.

Ces règles autorisent la lecture/écriture à toute personne authentifiée (même de façon
anonyme, ce qui est automatique) — adapté à un usage d'équipe interne sans gestion de
comptes individuels. Voir les commentaires dans `firestore.rules` pour aller plus loin.

## Étape 4 — Créer le dépôt GitHub et publier la page

1. Sur [github.com](https://github.com), crée un nouveau dépôt (**New repository**).
   - Il peut être **public** (GitHub Pages gratuit fonctionne avec un dépôt public ;
     un dépôt privé nécessite un abonnement GitHub Pro pour activer Pages).
2. Ajoute les fichiers de ce dossier au dépôt (`index.html`, `firestore.rules`, `README.md`) :

   ```bash
   git init
   git add index.html firestore.rules README.md
   git commit -m "Premier déploiement du tableau de bord qualité"
   git branch -M main
   git remote add origin https://github.com/TON-COMPTE/TON-DEPOT.git
   git push -u origin main
   ```

3. Sur GitHub : **Settings > Pages** (menu de gauche du dépôt).
   - Source : **Deploy from a branch**.
   - Branch : `main`, dossier `/ (root)`.
   - Sauvegarder.
4. Après une minute ou deux, GitHub affiche l'URL publique, du type :
   `https://ton-compte.github.io/ton-depot/`

C'est cette URL que tu partages avec Christophe, Abdelali et les autres managers.

## Étape 5 — Vérifier que ça fonctionne

Ouvre l'URL. En haut de la page, à droite, un indicateur de statut doit afficher
**« Connecté — partagé en ligne (Firebase) »**. Si à la place tu vois :

- **« Mode local — Firebase non configuré »** → la config de l'étape 2 n'a pas été
  correctement collée (vérifie qu'il ne reste pas `COLLE_TA_CLE_API_ICI`).
- **« Erreur de connexion Firebase »** → vérifie les règles Firestore (étape 3) et que
  l'authentification anonyme est bien activée (étape 1).

Pour vérifier le partage : ouvre l'URL sur deux navigateurs (ou un navigateur + le
téléphone), modifie une ligne sur l'un, elle doit apparaître sur l'autre en quelques
secondes.

---

## Mettre à jour le tableau de bord plus tard

Si tu modifies `index.html` (nouvelle fonctionnalité, correction), il suffit de repousser
le fichier :

```bash
git add index.html
git commit -m "Mise à jour du tableau de bord"
git push
```

GitHub Pages republie automatiquement la page en 1-2 minutes. **Les données stockées dans
Firestore ne sont pas affectées** par une mise à jour du code — seule l'apparence/les
fonctionnalités changent.

## Sauvegardes

Même avec Firebase, pense à exporter régulièrement une sauvegarde depuis l'onglet
**Extractions > Export pour régénérer le PowerPoint** (fichier `.json`) ou
**Générer le fichier Excel complet** — utile en cas de fausse manipulation, et c'est aussi
ce qui permet de renvoyer les données à Claude pour régénérer le PowerPoint.

## Limites connues

- Pas de système de compte individuel : tout le monde qui a l'URL peut tout modifier
  (pas de distinction "qui a fait quoi" au niveau des permissions, seulement dans le
  contenu des lignes elles-mêmes).
- Le PowerPoint ne se génère pas depuis la page (voir onglet Extractions) — il faut
  repasser par Claude avec l'export JSON.
- Firebase gratuit (offre Spark) a des quotas larges mais non illimités ; pour l'usage
  décrit ici (une équipe, quelques centaines de lignes), il n'y a pas de risque de les
  dépasser.
