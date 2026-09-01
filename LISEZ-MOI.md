# Vita — installer l'app sur ton téléphone

## 1. Mettre le site en ligne (nécessaire pour l'installer)

Pour qu'iPhone/Samsung acceptent de l'ajouter comme une vraie app (icône, plein écran, hors connexion), le site doit être servi en **https**. En local (fichier ouvert directement) l'installation ne fonctionnera pas.

Le plus simple, gratuit, sans rien installer :
1. Va sur **netlify.com/drop** (ou vercel.com, ou crée un dépôt GitHub + GitHub Pages).
2. Dépose les 5 fichiers du dossier (`index.html`, `manifest.json`, `service-worker.js`, `icon-192.png`, `icon-512.png`) — tous dans le même dossier, sans sous-dossier.
3. Tu obtiens une adresse du type `https://ton-site.netlify.app`.

## 2. Ajouter à l'écran d'accueil

**iPhone (Safari)** : ouvre le lien → bouton Partager (carré avec flèche) → « Sur l'écran d'accueil ».

**Samsung / Android (Chrome)** : ouvre le lien → menu ⋮ → « Ajouter à l'écran d'accueil » (ou une bannière d'installation apparaît automatiquement).

Une fois installée, l'icône s'ouvre en plein écran, sans barre d'adresse, comme une app native.

## 3. Fonctionnement hors connexion

Après la première ouverture (avec internet), l'interface et tout ce que tu as déjà enregistré (tâches, budget, documents, mood…) restent consultables et modifiables sans connexion. Les nouvelles données que tu ajoutes hors ligne sont aussi sauvegardées, et resteront là au retour de la connexion — tout est stocké uniquement sur ton téléphone, dans le navigateur, il n'y a pas de serveur derrière.

**Important** : ces données ne sont pas automatiquement sauvegardées ailleurs. Si tu changes de téléphone ou effaces les données du navigateur, tu les perds. Utilise le bouton ⚙️ en haut de l'app pour exporter une sauvegarde (fichier .json) de temps en temps, et pouvoir la réimporter si besoin.

## 4. Ce qui est inclus

- Accueil avec résumé du jour et prochaines échéances
- Études & alternance, Logement, Courses, Sport & santé, Relations, Voyages, Objectifs, Inventaire
- Budget (revenus/dépenses, solde, répartition par catégorie)
- Skincare & glow up (routines matin/soir, suivi produits)
- Mood (humeur du jour + journal)
- Documents (ajout de fichiers/photos, consultables hors ligne)
- Centre de rappels qui regroupe toutes les échéances (devoirs, factures, anniversaires, rachats produits…)
- Sauvegarde/restauration manuelle des données

## 5. Activer les comptes et la messagerie (Firebase — gratuit)

Sans cette étape, l'app fonctionne quand même : un bouton "Continuer sans compte" apparaît et tout le reste (études, budget, documents…) marche normalement. Voici comment activer les comptes + la messagerie entre deux personnes.

**A. Créer le projet**
1. Va sur [console.firebase.google.com](https://console.firebase.google.com), connecte-toi avec un compte Google.
2. "Ajouter un projet" → donne-lui un nom (ex. `vita-app`) → suis les étapes (tu peux désactiver Google Analytics, pas nécessaire).

**B. Activer l'authentification**
1. Dans le menu de gauche : **Build → Authentication → Get started**.
2. Onglet "Sign-in method" → active **Email/Password**.

**C. Créer la base de données (messagerie)**
1. Menu de gauche : **Build → Firestore Database → Create database**.
2. Choisis "Start in production mode", puis une région proche (ex. `eur3 (europe-west)`).
3. Une fois créée, va dans l'onglet **Rules** et remplace tout par :
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users_by_email/{emailId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.resource.data.uid == request.auth.uid;
    }
    match /channels/{channelId} {
      allow read, update: if request.auth != null && request.auth.uid in resource.data.members;
      allow create: if request.auth != null && request.auth.uid in request.resource.data.members;
      match /messages/{messageId} {
        allow read: if request.auth != null && request.auth.uid in get(/databases/$(database)/documents/channels/$(channelId)).data.members;
        allow create: if request.auth != null && request.auth.uid in get(/databases/$(database)/documents/channels/$(channelId)).data.members && request.resource.data.senderId == request.auth.uid;
      }
    }
  }
}
```
4. Clique **Publish**.

**D. Récupérer la configuration et la coller dans l'app**
1. Menu de gauche → icône ⚙️ à côté de "Project Overview" → **Project settings**.
2. Descends jusqu'à "Your apps" → clique l'icône **</>** (Web) → donne un nom → "Register app".
3. Firebase affiche un objet `firebaseConfig = {...}` : copie-le.
4. Ouvre le fichier **firebase-config.js** (fourni avec l'app) et remplace les valeurs `"REMPLACE_MOI"` par celles copiées.
5. Renvoie ce fichier sur GitHub (remplace l'ancien), attends que GitHub Pages redéploie.

**E. Utilisation**
- Chaque personne crée son compte (email + mot de passe), vérifie son email via le lien reçu, puis choisit un code à 4 chiffres pour déverrouiller l'app rapidement ensuite sur son téléphone.
- Pour discuter avec quelqu'un, cette personne doit **déjà avoir créé son compte Vita** — ensuite, onglet 💬 Messages → "Nouvelle conversation" → son email.
- Le code à 4 chiffres n'est qu'un verrou rapide sur l'appareil, pas la sécurité du compte (celle-ci repose sur le mot de passe) — "Code oublié ?" permet d'en redéfinir un sans perdre l'accès au compte.
