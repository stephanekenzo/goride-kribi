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

## Style visuel

L'interface reprend les codes visuels des applications VTC connues (Uber,
Yango) : fond **noir**, accent rouge-orangé, écran d'accueil en grille de
services, parcours de réservation en écrans successifs. Le nom et le
logo restent les vôtres ("GoRide Kribi") — aucune marque tierce n'est
reproduite.

## Les services proposés (écran d'accueil)

| Service | État | Ce que ça fait |
|---|---|---|
| 🚗 Course | Fonctionnel | Réservation de chauffeur, comme avant |
| 🏍️ Moto | Fonctionnel | Même parcours, tarif à 60% du prix voiture |
| 📦 Colis | Fonctionnel | Envoi de paquet, un chauffeur récupère et livre (formulaire dédié : point de récupération, livraison, description) |
| 🧳 Voyage | Fonctionnel | Raccourci direct vers les tarifs Kribi-Douala / Kribi-Yaoundé |
| 🔑 Location | Fonctionnel (catalogue) | Vrai catalogue de véhicules (Berline, SUV, Pickup 4x4, 4x4 Luxe...) géré dans l'admin — nom, type, prix/jour, caution, avec ou sans chauffeur, propriétaire. Le client parcourt, choisit, indique ses dates ; la demande arrive dans l'admin pour confirmation (pas de paiement ni de calendrier de disponibilité automatique pour l'instant). |
| 🧭 Explorer | Maquette + prise de contact | Formulaire simple, demande transmise à l'admin |
| 🛍️ Restos & Boutiques | Fonctionnel (appel + livraison) | Liste de vrais restaurants de Kribi (nom, spécialité, téléphone réel). Le client appelle pour commander, puis un bouton "Demander une livraison" ouvre directement le formulaire Colis avec le restaurant pré-rempli comme point de récupération — un chauffeur va chercher et livre. |

Les demandes "Explorer" atterrissent dans l'onglet **"Demandes"** de
l'admin. Les demandes de location apparaissent aussi là, avec le
véhicule demandé.

## Calcul du prix par la distance

Quand la position du client est connue (géolocalisation) **et** que la
destination choisie est l'une des suggestions réelles de Kribi (hôtels,
restaurants, sites — avec coordonnées trouvées sur Google Maps), le prix
se calcule automatiquement : **distance × prix/km**, avec un minimum
garanti. Par défaut 500 FCFA/km — modifiable dans l'admin, onglet
Tarifs. La moto revient à 60% de ce prix.

Si la destination est tapée librement (pas dans la liste) ou si la
position n'est pas disponible, l'app retombe sur les tarifs fixes par
zone (comme avant), pour que la réservation marche toujours.

## Le catalogue de véhicules en location

Géré dans l'admin, onglet **"Véhicules"** (même principe que les
chauffeurs : ajouter, modifier, désactiver, supprimer). Pour chaque
véhicule : nom, type (Berline / SUV / Pickup 4x4 / 4x4 Luxe / Minibus),
prix/jour, caution, avec ou sans chauffeur, et propriétaire (utile si
ce ne sont pas vos propres véhicules mais ceux de partenaires qui vous
confient leur location).

Tant qu'aucun véhicule n'est ajouté dans l'admin, l'app affiche 4
exemples par défaut (Berline 30 000, SUV 50 000, Toyota Hilux 70 000,
Toyota Prado 100 000 FCFA/jour) — remplacez-les par votre vrai
catalogue dès que possible.

## Comment ça marche

- **Client** : à partir de l'écran d'accueil (grille de services), le
  parcours "Course"/"Moto" est en 3 écrans : Recherche (départ/
  destination, avec les vraies destinations de Kribi en raccourcis) →
  Détails (voiture/moto, prix calculé ou zones de repli, attribution du
  chauffeur) → Suivi. "Colis" et les demandes spéciales ont chacun leur
  propre formulaire court.

  Par défaut ("Automatique"), la demande part vers le chauffeur
  disponible le plus proche ; s'il ne répond pas sous 40 secondes, elle
  passe au suivant, jusqu'à épuisement (elle est alors ouverte à tous).
  Le client peut aussi choisir "Choisir moi-même" pour sélectionner un
  chauffeur précis dans la liste triée par distance.
- **Chauffeur** : connexion téléphone + PIN. Tant qu'il est "disponible",
  sa position GPS est envoyée à Firestore toutes les ~20 secondes (utilisée
  pour le tri par proximité, pas pour un affichage cartographique). Il
  voit en priorité les demandes qui lui sont directement adressées
  (cascade), puis les demandes ouvertes à tous, triées de la plus proche
  à la plus lointaine, avec un badge Colis/Moto quand c'est le cas ;
  alerte sonore + notification navigateur à l'arrivée d'une nouvelle
  demande.
- **Admin** : onglets Courses, Demandes (spéciales), Chauffeurs, Tarifs
  (calcul au km + zones de repli). Bouton "Libérer à tous les
  chauffeurs" pour débloquer une demande restée bloquée sur un
  chauffeur.

Cette version n'utilise volontairement **aucune carte interactive** —
plusieurs essais (OpenStreetMap, puis Mapbox) n'ont pas donné un rendu
fiable dans le contexte de déploiement. La distance et le tri par
proximité fonctionnent toujours (calcul GPS en coulisses), seul
l'affichage visuel sur une carte a été retiré pour garantir un
fonctionnement stable. Ça pourra être réintroduit plus tard comme
amélioration séparée, une fois la cause du problème identifiée.

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
