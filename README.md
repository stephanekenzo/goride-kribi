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

- **Client** : deux types de trajet.
  - **En ville** : une carte interactive s'ouvre, avec un point rose clair
    (départ, positionné automatiquement puis ajustable à la main) et un
    point rose foncé que le client place en touchant la carte
    (destination). Le prix est calculé automatiquement sur la **distance
    réelle par la route** (API gratuite OSRM, sans clé) : prix de prise
    en charge + prix/km, avec un minimum garanti — tout est réglable dans
    l'admin.
  - **Longue distance** (Kribi-Douala, Kribi-Yaoundé) : tarif fixe comme
    avant, pas besoin de carte.

  Dans les deux cas, la demande est automatiquement envoyée au chauffeur
  disponible le plus proche ; s'il ne répond pas sous 40 secondes, elle
  passe au suivant, jusqu'à épuisement (elle est alors ouverte à tous).
  Une fois la course acceptée, une **carte en direct** affiche la
  position du chauffeur qui se met à jour automatiquement.
- **Chauffeur** : connexion téléphone + PIN. Tant qu'il est "disponible",
  sa position GPS est envoyée à Firestore toutes les ~20 secondes. Il voit
  en priorité les demandes qui lui sont directement adressées (cascade),
  puis les demandes ouvertes à tous, triées de la plus proche à la plus
  lointaine, avec la distance du trajet affichée ; alerte sonore +
  notification navigateur à l'arrivée d'une nouvelle demande.
- **Admin** : vue de toutes les courses (avec un bouton "Libérer à tous
  les chauffeurs" pour débloquer une demande restée bloquée sur un
  chauffeur), gestion des chauffeurs (ajout, modification, désactivation,
  suppression), et grille tarifaire — les 3 paramètres du calcul en
  ville (prise en charge, prix/km, minimum) + les 2 tarifs fixes longue
  distance.

### Calcul de distance en ville : comment ça marche

Le calcul appelle l'API gratuite **OSRM** (router.project-osrm.org, sans
clé) pour obtenir la vraie distance par la route entre les deux points
placés sur la carte. C'est un serveur de démonstration public — gratuit
et sans carte bancaire, mais pas garanti à 100% en disponibilité. Si
l'appel échoue, l'application retombe automatiquement sur la distance à
vol d'oiseau majorée de 30% (affichée comme "estimation"), pour que le
client ait toujours un prix.

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
