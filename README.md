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

## Comment ça marche (modèle façon Yango)

- **Client** : la localisation se lance automatiquement à l'ouverture.
  Le client choisit une zone (tarif automatique), indique sa destination,
  son nom et son téléphone. Par défaut ("Automatique"), la demande est
  envoyée au chauffeur disponible le plus proche ; s'il ne répond pas
  sous 40 secondes, elle passe automatiquement au suivant, puis au
  suivant, jusqu'à épuisement (elle est alors ouverte à tous). Le client
  peut aussi choisir "Choisir moi-même" pour sélectionner un chauffeur
  précis dans la liste triée par distance. Une fois la course acceptée,
  une **carte en direct** (OpenStreetMap, sans clé API) affiche la
  position du chauffeur qui se met à jour automatiquement.
- **Chauffeur** : connexion téléphone + PIN. Tant qu'il est "disponible",
  sa position GPS est envoyée à Firestore toutes les ~20 secondes. Il voit
  en priorité les demandes qui lui sont directement adressées (cascade),
  puis les demandes ouvertes à tous, triées de la plus proche à la plus
  lointaine ; alerte sonore + notification navigateur à l'arrivée d'une
  nouvelle demande.
- **Admin** : vue de toutes les courses (avec un bouton "Libérer à tous
  les chauffeurs" pour débloquer une demande restée bloquée sur un
  chauffeur, par exemple si le client a fermé son navigateur avant la fin
  de la cascade), gestion des chauffeurs (ajout, modification,
  désactivation, suppression), grille tarifaire.

### Limite importante de la cascade automatique

La cascade (passer au chauffeur suivant après 40 secondes) est pilotée
par le **navigateur du client**, pas par un serveur — ce projet n'utilise
aucun serveur pour rester simple à déployer (pas de Cloud Functions). Si
le client ferme la page avant qu'un chauffeur accepte, la cascade s'arrête
et la demande reste assignée au dernier chauffeur contacté. C'est pour
ça que le bouton "Libérer à tous les chauffeurs" existe dans l'admin —
utilisez-le si une course semble bloquée sans réponse.

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
