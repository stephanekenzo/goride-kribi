# GoRide Kribi — Version fichier unique (comme Fidélie Hôtel / SPIHT-Océan)

Même architecture que vos autres apps : pages HTML autonomes + Firebase +
Vercel, sans build, sans npm, sans variables d'environnement à configurer
sur Vercel. Vous poussez les fichiers tels quels sur GitHub, Vercel les
sert directement.

3 pages :
- `index.html` — réservation client
- `chauffeur.html` — connexion + tableau de bord chauffeur (GPS, alertes)
- `admin.html` — gestion chauffeurs / courses / tarifs

---

## 1. Créer le projet Firebase

1. https://console.firebase.google.com → nouveau projet (ex. "goride-kribi").
2. **Compilation > Firestore Database** → Créer une base, mode **production**.
3. Onglet **Règles** → collez le contenu de `firestore.rules` → Publier.
4. Onglet **Index** → créez un index composite :
   - Collection : `courses`
   - Champs : `statut` (Croissant), `chauffeurCibleId` (Croissant),
     `createdAt` (Décroissant)
   - Portée : Collection
5. **Paramètres du projet > Général** → section "Vos applications" → icône
   Web (`</>`) → donnez un nom → copiez les 6 valeurs de `firebaseConfig`.

## 2. Coller votre config Firebase dans les 3 fichiers

Ouvrez `index.html`, `chauffeur.html` et `admin.html`. Dans chacun, cherchez
ce bloc (identique dans les trois) et remplacez les valeurs :

```js
const firebaseConfig = {
  apiKey: "VOTRE_API_KEY",
  authDomain: "VOTRE_PROJET.firebaseapp.com",
  projectId: "VOTRE_PROJET",
  storageBucket: "VOTRE_PROJET.appspot.com",
  messagingSenderId: "VOTRE_SENDER_ID",
  appId: "VOTRE_APP_ID",
};
```

Ces valeurs ne sont pas secrètes (c'est la config publique du client
Firebase) — la sécurité réelle vient des règles Firestore, pas de ce
fichier.

Dans `admin.html`, changez aussi le mot de passe admin :

```js
const ADMIN_PASSWORD = "changez-moi";
```

## 3. Mettre sur GitHub, puis déployer sur Vercel

1. Créez un repository GitHub, téléversez les fichiers de ce dossier
   (glisser-déposer depuis github.com fonctionne très bien depuis un
   téléphone).
2. Sur https://vercel.com → "Add New Project" → importez le repository.
3. Framework preset : laissez sur **Other** (aucun build nécessaire).
4. "Deploy". Après 30-60 secondes, votre site est en ligne.

Aucune variable d'environnement à ajouter sur Vercel — tout est déjà dans
les fichiers HTML.

## 4. Ajouter vos chauffeurs

`votre-site.vercel.app/admin.html` → mot de passe → onglet "Chauffeurs" →
"Ajouter un chauffeur" (nom, téléphone, véhicule, code PIN à 4 chiffres).
Le chauffeur se connecte ensuite sur `votre-site.vercel.app/chauffeur.html`
avec ce numéro et ce code.

## 5. Ajuster la grille tarifaire

Onglet "Tarifs" du panel admin — modifiable à tout moment.

---

## Comment ça marche

- **Client** : choisit une zone (tarif automatique), remplit départ/
  destination/nom/téléphone, envoie à tous les chauffeurs disponibles ou
  choisit un chauffeur précis dans une liste triée par distance (position
  partagée volontairement). Suivi en temps réel jusqu'à l'acceptation.
- **Chauffeur** : connexion téléphone + PIN. Tant qu'il est "disponible",
  sa position GPS est envoyée à Firestore toutes les ~20 secondes. Les
  demandes générales sont triées de la plus proche à la plus lointaine ;
  alerte sonore + notification navigateur à l'arrivée d'une nouvelle
  demande.
- **Admin** : vue de toutes les courses, gestion des chauffeurs (ajout,
  modification, désactivation, suppression), grille tarifaire.

## Notes de sécurité (comme pour vos autres apps)

- Les règles Firestore fournies sont ouvertes (lecture/écriture libres)
  pour fonctionner sans configuration complexe — correct pour démarrer
  avec une dizaine de chauffeurs de confiance, à renforcer si l'app grandit.
- Le mot de passe admin et les codes PIN chauffeurs sont une protection
  basique côté client, pas une authentification serveur.
- La géolocalisation exige HTTPS (Vercel le fournit automatiquement).

## Limites connues de cette V1

- Notifications uniquement si le chauffeur a la page ouverte ou en
  arrière-plan récent (pas de push complet — demanderait Firebase Cloud
  Messaging).
- Distance à vol d'oiseau (formule de Haversine), pas un temps de trajet
  routier réel.
- Pas de paiement intégré — réglé en direct avec le chauffeur comme
  aujourd'hui.
