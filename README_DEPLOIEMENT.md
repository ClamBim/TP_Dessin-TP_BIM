# Déploiement — Suivi pédagogique TEBDP en ligne

**Architecture :** GitHub Pages héberge l'application (`index.html`) · Supabase gère les comptes et stocke les données partagées.
Supabase n'héberge pas de pages web : c'est le duo GitHub Pages (front) + Supabase (backend) qui rend l'application accessible en ligne à toute l'équipe et aux alternants.

**Projet Supabase :** `https://ovbgtmfyktlliqeiereg.supabase.co` (la clé publique `sb_publishable_…` est déjà intégrée dans `index.html` ; elle est conçue pour être exposée — la sécurité repose sur les politiques RLS du script SQL, pas sur le secret de cette clé).

---

## Étape 1 — Préparer Supabase (5 min)

1. Ouvrez votre projet → **SQL Editor** → *New query*.
2. Collez tout le contenu de **`supabase_setup.sql`** → **Run**.
   La dernière requête doit afficher une ligne avec `t` (true) dans les 5 colonnes de contrôle.
3. **Authentication → Sign In / Up** : vérifiez que le provider **Email** est activé (il l'est par défaut).
   - *Confirm email* **activé** (recommandé) : chaque compte doit cliquer un lien de confirmation reçu par e-mail avant la première connexion.
   - Pour une mise en route sans e-mails de confirmation (journée de positionnement), désactivez *Confirm email* — réactivable ensuite.
4. **Authentication → URL Configuration** :
   - *Site URL* : l'adresse GitHub Pages obtenue à l'étape 2 (ex. `https://VOTRE-COMPTE.github.io/suivi-tebdp/`).
   - Ajoutez la même adresse dans *Redirect URLs* (nécessaire pour « Mot de passe oublié »).

## Étape 2 — Publier sur GitHub Pages (5 min)

1. Sur github.com : **New repository** → nom `suivi-tebdp` → **Public** → *Create*.
2. **Add file → Upload files** → déposez **`index.html`** → *Commit*.
3. **Settings → Pages** → *Source* : « Deploy from a branch » → branche `main`, dossier `/ (root)` → *Save*.
4. Après 1–2 min, l'application est en ligne : `https://VOTRE-COMPTE.github.io/suivi-tebdp/`.
   → Reportez cette adresse dans Supabase (étape 1.4).

*(Le dépôt étant public, n'importe qui peut voir le code de la page — mais aucune donnée : celles-ci vivent dans Supabase, derrière l'authentification et les politiques RLS.)*

## Étape 3 — Migrer vos données actuelles (2 min)

1. Dans votre application **locale** (v7.1) : onglet **06 · Sauvegarde & export** → **Exporter la sauvegarde (.json)**.
2. Sur l'application **en ligne** : créez votre compte `…@prof.gretacfa-montpellier.fr`, connectez-vous → onglet **06** → **Restaurer une sauvegarde (.json)**.
3. Les PDF ajoutés dans Ressources ne sont pas inclus dans le .json : ré-ajoutez-les une fois en ligne (ils seront alors stockés dans Supabase et visibles de tous).

---

## Règles d'accès

| Adresse de connexion | Rôle | Accès |
|---|---|---|
| `…@prof.gretacfa-montpellier.fr` | Formateur | **Menu complet** + écriture des données + validation des comptes |
| `laurent.colombero@lecnam.net` | Formateur (compte nommé) | Identique aux formateurs |
| `…@gretacfa-montpellier.fr` (ou tout sous-domaine : `@etu.…`, etc.) | Étudiant | **« Vue par thématique » uniquement**, en lecture seule, **après validation par un formateur** |
| Toute autre adresse | — | Inscription et connexion refusées (application **et** serveur) |

La restriction est appliquée à trois niveaux : l'écran de connexion (message clair), un trigger sur la création de comptes (impossible de s'inscrire hors domaine), et les politiques RLS (même un étudiant « bricoleur » passant par l'API ne peut ni écrire, ni lire sans compte GRETA **validé**).

## Validation des comptes étudiants (v2)

1. L'étudiant crée son compte avec son adresse GRETA-CFA → message « **en attente de validation par votre formateur** » ; il ne voit aucune donnée tant que vous n'avez pas statué (verrouillé aussi côté serveur).
2. À votre connexion (formateur), une carte dorée « **Comptes étudiants à valider** » apparaît en tête du **Récapitulatif** : adresse, date et heure de la demande, boutons **Valider l'accès** / **Refuser**.
3. Une fois validé, l'étudiant actualise sa page (F5) et entre directement.

Astuce : avec ce circuit de validation, vous pouvez désactiver *Confirm email* dans Supabase (Authentication → Sign In / Up) — l'étudiant n'a alors plus de lien de confirmation à cliquer, votre validation reste l'unique porte d'entrée.

Notification par e-mail à chaque demande : Supabase seul n'envoie pas d'e-mail aux formateurs. Si vous la souhaitez, il faut ajouter une Edge Function reliée à un service d'envoi (ex. Resend, gratuit jusqu'à 100 e-mails/jour) — demandez-la à Claude avec votre clé API Resend.

## Onglet 07 · Administration (v3, formateurs uniquement)

- **Liste de tous les comptes** : adresse, rôle (formateur/étudiant), statut (validé / en attente / refusé), date de création, validé par qui et quand.
- **Valider / Suspendre** l'accès d'un étudiant à tout moment (une suspension le déconnecte de fait des données : la lecture est re-vérifiée côté serveur).
- **Nouveau code d'accès** : génère un code type `TEBDP-XXXXXXXX` (modifiable avant envoi), le pose comme mot de passe de l'étudiant, le copie dans votre presse-papiers pour transmission, et déconnecte ses sessions ouvertes.
- **Supprimer** définitivement un compte étudiant (double confirmation). Les fiches alternants et passations du suivi ne sont pas touchées — seule la connexion est retirée.
- Garde-fous côté serveur : ces trois actions sont refusées sur les comptes formateurs et aux non-formateurs, quelle que soit la manière dont l'API est appelée.

Après mise à jour : ré-exécutez `supabase_setup.sql` (v3, ré-exécutable sans risque) puis remplacez `index.html` dans le dépôt GitHub.

## Création des comptes étudiants par le formateur (v4 · application v8.3)

Pour les étudiants qui ne reçoivent pas l'e-mail de confirmation, le formateur crée lui-même les comptes depuis l'onglet **07 · Administration**, carte « **Créer des comptes étudiants** » :

- **Saisie manuelle** : une ou plusieurs adresses (une par ligne, ou séparées par `;` ou `,`) → « Ajouter à la liste ».
- **Import Excel ou CSV** : une ligne par étudiant ; la colonne des adresses est repérée automatiquement, colonnes `Nom` / `Prénom` facultatives (modèle téléchargeable depuis la carte).
- **Aperçu avant création** : chaque adresse est contrôlée (hors domaine GRETA, adresse formateur, doublon, compte déjà existant, inscription non confirmée à débloquer) ; un code `TEBDP-XXXXXXXX` est proposé pour chacune, modifiable.
- **Création** : compte créé **adresse confirmée et accès validé**, sans aucun e-mail envoyé. Un étudiant déjà inscrit mais jamais confirmé est **débloqué** (adresse confirmée, accès validé, code posé). Un compte déjà actif n'est pas modifié.
- **Récapitulatif** : téléchargement Excel (adresses, codes, adresse de l'application, consignes de connexion) ou copie dans le presse-papiers. Les codes ne sont stockés nulle part en clair : téléchargez le récapitulatif avant de quitter la page.

Dans la liste des utilisateurs : badge « **Non confirmée** » + bouton « **Débloquer** » pour les inscriptions restées sans confirmation, et date de dernière connexion de chaque compte.

**Mise en place :** exécutez **`supabase_v4_creation_comptes.sql`** dans Supabase → SQL Editor (après le script v3 ; ré-exécutable sans risque — la dernière requête doit afficher `t` dans les 2 colonnes), puis remplacez `index.html` dans le dépôt GitHub. Garde-fous côté serveur : fonctions réservées aux formateurs, adresses GRETA étudiantes uniquement, code de 8 caractères minimum.

> Si le domaine e-mail réel des étudiants n'est pas un sous-domaine de `gretacfa-montpellier.fr`, dites-le-moi : deux lignes à changer (constantes `DOMAINE_GRETA` dans `index.html` et regex dans le SQL).

## Tests passés par les étudiants + vue individuelle (v5 · application v8.4)

**Côté étudiant (onglet 02)**
- Carte « **Mes tests de positionnement** » : les 6 tests (AutoCAD, Revit · Dessin · Métré, Normes, Énergie & thermique, Permis de construire, Technologie), plus le bouton 🧪 de la thématique sélectionnée.
- À la génération du rapport, le résultat est **transmis automatiquement** au formateur (table `depots_tests`) ; l'étudiant peut toujours télécharger son rapport. Un rapport généré deux fois n'est transmis qu'une fois.
- Liste « **Mes résultats transmis** » : En attente d'intégration / Intégré à votre suivi / Non retenu.
- L'étudiant ne voit plus que **ses propres résultats** : le serveur ne lui renvoie que sa fiche (fonction `tebdp_donnees_etudiant`) et la lecture directe des données de suivi est réservée aux formateurs — même via l'API.

**Rattachement compte ↔ fiche alternant**
1. adresse e-mail renseignée sur la fiche (nouveau champ « Adresse e-mail (compte étudiant) », rempli automatiquement à la première intégration) ;
2. à défaut, correspondance `prenom.nom@…` ↔ « NOM Prénom » (accents, tirets et ordre ignorés).
Si un étudiant ne voit pas ses résultats, renseignez son adresse sur sa fiche (onglet 03).

**Côté formateur (Récapitulatif)**
- Carte « **Résultats de tests transmis par les étudiants** » : test, score, adresse, nom saisi, fiche cible (existante ou nouvelle) → **Intégrer au suivi** (étape déterminée automatiquement), **Ignorer**, **Tout intégrer**.

**Mise en place :** exécutez **`supabase_v5_tests_etudiants.sql`** dans Supabase → SQL Editor (après v3 et v4 ; ré-exécutable sans risque — la dernière requête doit afficher `t` dans les 4 colonnes), puis remplacez `index.html` dans le dépôt GitHub.
Sans le script, l'application v8.4 fonctionne quand même : la vue est filtrée côté navigateur (moins sûr) et l'étudiant est invité à télécharger son rapport.
**Important :** si vous ré-exécutez un jour `supabase_setup.sql`, ré-exécutez ensuite le script v5.

## Bon à savoir

- **Écritures simultanées** : les données forment une sauvegarde partagée « dernier écrit gagne ». Si deux formateurs saisissent en même temps, le dernier enregistrement écrase l'autre — actualisez la page (F5) avant une session de saisie, et exportez un .json avant les grosses opérations.
- **Mot de passe oublié** : lien sur l'écran de connexion → e-mail de réinitialisation (nécessite l'étape 1.4).
- **Mises à jour de l'application** : remplacez `index.html` dans le dépôt GitHub (Upload files → écraser) ; les données ne sont pas touchées.
- **Sauvegardes** : l'export .json de l'onglet 06 reste votre filet de sécurité — un export hebdomadaire est une bonne habitude.
- Les 6 tests de positionnement sont intégrés : lancés par les formateurs (onglets 02 et 05) ou par les étudiants eux-mêmes depuis leur onglet 02 (v5) ; les fichiers de test autonomes (HTML) continuent de fonctionner hors ligne, indépendamment de Supabase.
