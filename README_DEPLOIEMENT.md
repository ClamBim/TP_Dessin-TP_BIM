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
| `…@prof.gretacfa-montpellier.fr` | Formateur | **Menu complet** + écriture des données |
| `…@gretacfa-montpellier.fr` (ou tout sous-domaine : `@etu.…`, etc.) | Étudiant | **« Vue par thématique » uniquement**, en lecture seule |
| Toute autre adresse | — | Inscription et connexion refusées (application **et** serveur) |

La restriction est appliquée à trois niveaux : l'écran de connexion (message clair), un trigger sur la création de comptes (impossible de s'inscrire hors domaine), et les politiques RLS (même un étudiant « bricoleur » passant par l'API ne peut ni écrire, ni lire sans compte GRETA).

> Si le domaine e-mail réel des étudiants n'est pas un sous-domaine de `gretacfa-montpellier.fr`, dites-le-moi : deux lignes à changer (constantes `DOMAINE_GRETA` dans `index.html` et regex dans le SQL).

## Bon à savoir

- **Écritures simultanées** : les données forment une sauvegarde partagée « dernier écrit gagne ». Si deux formateurs saisissent en même temps, le dernier enregistrement écrase l'autre — actualisez la page (F5) avant une session de saisie, et exportez un .json avant les grosses opérations.
- **Mot de passe oublié** : lien sur l'écran de connexion → e-mail de réinitialisation (nécessite l'étape 1.4).
- **Mises à jour de l'application** : remplacez `index.html` dans le dépôt GitHub (Upload files → écraser) ; les données ne sont pas touchées.
- **Sauvegardes** : l'export .json de l'onglet 06 reste votre filet de sécurité — un export hebdomadaire est une bonne habitude.
- Les 5 tests de positionnement restent intégrés et utilisables par les formateurs ; les fichiers de test autonomes (HTML) continuent de fonctionner hors ligne, indépendamment de Supabase.
