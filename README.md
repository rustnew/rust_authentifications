Parfait. T'as un projet **complet d'authentification en Rust avec Axum et Shuttle**, prêt pour le web. Tu es à deux doigts de tout faire tourner. Voici une **explication détaillée de chaque partie**, et comment le **lancer proprement en local**.

---

## 🚀 OBJECTIF DU PROJET

Ce projet web :
- gère **l’inscription, la connexion, la suppression** et la **consultation des utilisateurs**
- fonctionne avec un **frontend HTML (Tera)** + un backend **Axum**
- gère les sessions utilisateurs avec **cookies**
- utilise une base de données **PostgreSQL** via **SQLx**
- se déploie avec **Shuttle**

---

## 🧠 ARCHITECTURE DU CODE

### 1. **Fichiers `mod`:**
```rust
mod errors;
mod authentication;
mod utils;
```

Ce sont des modules internes :
- `errors.rs`: contient tes erreurs customisées.
- `authentication.rs`: toute la logique d'inscription, login, token, suppression.
- `utils.rs`: probablement des helpers pour parse `Multipart`, cookies, etc.

---

### 2. **Main: Démarrage du serveur**

```rust
#[shuttle_runtime::main]
async fn server(
  #[shuttle_shared_db::Postgres] pool: PgPool
) -> ShuttleAxum {
```

- `#[shuttle_runtime::main]`: ton point d’entrée pour Shuttle.
- `#[shuttle_shared_db::Postgres]`: injecte automatiquement un **pool PostgreSQL**.
- `pool.execute(include_str!("../schema.sql"))`: exécute le fichier SQL au lancement (création des tables).

Il retourne un `Router` défini dans `get_router`.

---

### 3. **Router: Les routes HTTP**

```rust
pub fn get_router(database: Database) -> Router { ... }
```

Il expose toutes les **routes web** :

| Route                 | Action                              |
|----------------------|-------------------------------------|
| `/`                  | Page d’accueil                      |
| `/signup`            | Inscription (GET+POST)              |
| `/login`             | Connexion (GET+POST)                |
| `/logout`            | Déconnexion                         |
| `/delete`            | Suppression de compte               |
| `/me`                | Redirige vers `/user/:username`     |
| `/user/:username`    | Page profil utilisateur             |
| `/users`             | Liste des utilisateurs              |
| `/styles.css`        | Feuille de style CSS                |

---

### 4. **Templates Tera**
Ils sont chargés avec :

```rust
include_str!("../templates/...")
```

> Il faut donc que tu aies un dossier `templates/` **à la racine** avec les fichiers `.html`.

---

### 5. **Base de données PostgreSQL**

Tu utilises SQLx, via Shuttle :

- La base est connectée automatiquement avec `#[shuttle_shared_db::Postgres]`.
- Tu exécutes le fichier `schema.sql` à chaque lancement.

Le type :

```rust
type Database = sqlx::PgPool;
```

Tu fais des requêtes comme :

```rust
sqlx::query_as(QUERY).bind(...).fetch_optional(&database).await;
```

---

### 6. **Sécurité / Authentification**

Ton middleware :

```rust
.layer(middleware::from_fn(move |req, next| {
    auth(req, next, middleware_database.clone())
}))
```

→ Ce middleware injecte `AuthState` dans toutes les requêtes : il sert à savoir **qui est connecté**.

Les routes comme `/delete`, `/me`, `/user/:username` utilisent ça pour protéger l’accès.

---

### 7. **Génération de mot de passe**

Tu utilises `pbkdf2` + `ChaCha8Rng` → pour générer des **hashs de mot de passe** sécurisés, avec un **générateur de nombre aléatoire** `Arc<Mutex<ChaCha8Rng>>`.

---

## 📦 COMMENT LANCER LE PROJET

### ✅ 1. Installer Shuttle CLI si pas encore fait

```bash
cargo install cargo-shuttle
```

---

### ✅ 2. Créer un compte Shuttle

```bash
cargo shuttle login
```

---

### ✅ 3. Crée un fichier `Shuttle.toml` à la racine (s’il n'existe pas) :

```toml
name = "rustauth"
```

---

### ✅ 4. Vérifie le schéma SQL

Fichier `schema.sql` dans le dossier parent (`../schema.sql`) :
- Crée une table `users`
- Ex:  
```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL
);
```

Si tu ne l’as pas, crée-le.

---

### ✅ 5. Lancer le projet en local

```bash
cargo shuttle run
```

➡️ Si tout se passe bien, tu verras un message du style :

```
🚀 App running at http://127.0.0.1:8000
```

Puis, ouvre ton navigateur : `http://127.0.0.1:8000`

---

## 🧱 PRÉREQUIS POUR QUE ÇA MARCHE

- Un fichier `schema.sql`
- Un dossier `templates/` avec :
  - `base.html`
  - `index.html`
  - `login.html`
  - `signup.html`
  - `users.html`
  - `user.html`
- Un dossier `public/` avec :
  - `styles.css`

---

## 💥 Résumé

| Élément         | Détail                                                                 |
|----------------|------------------------------------------------------------------------|
| Framework       | Axum (web) + Shuttle (déploiement et dev local)                       |
| Base de données | PostgreSQL via SQLx                                                   |
| Templates       | Tera                                                                   |
| Authentification| Cookies, `pbkdf2`, token, suppression                                 |
| Lancement local | `cargo shuttle run`                                                   |
| Déploiement     | `cargo shuttle deploy` *(optionnel)*                                  |

---

Si tu veux, je peux aussi :
- te générer un exemple de `schema.sql`
- ou te vérifier que tes fichiers `templates` sont bons
- ou te faire une version **encore plus commentée** du code

Tu veux qu’on aille plus loin ?
