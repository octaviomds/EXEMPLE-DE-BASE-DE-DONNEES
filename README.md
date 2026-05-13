## Présentation

phpMyAdmin est une interface web permettant de gérer des bases de données MySQL sans passer par la ligne de commande. Il est intégré à MAMP et accessible via le navigateur.

Accès par défaut avec MAMP :

```
http://localhost:8888/phpMyAdmin
```

Identifiants par défaut :

- Utilisateur : `root`
- Mot de passe : `root`

---

## Structure de la base de données

Le projet utilise une base de données MySQL organisée en tables relationnelles.



### Créer une base de données

Dans phpMyAdmin, cliquer sur **Nouvelle base de données**, saisir un nom puis valider.

Exemple en SQL :

```sql
CREATE DATABASE ma_base CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Créer une table

```sql
CREATE TABLE utilisateurs (
    id        INT AUTO_INCREMENT PRIMARY KEY,
    nom       VARCHAR(100) NOT NULL,
    email     VARCHAR(150) NOT NULL UNIQUE,
    mot_passe VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```


## CRUD - Opérations de base

Le CRUD regroupe les quatre opérations fondamentales sur une base de données : Create, Read, Update, Delete.


### Create - Insérer des données

Via phpMyAdmin : onglet **Insérer**, remplir les champs et valider.

En SQL :

```sql
INSERT INTO utilisateurs (nom, email, mot_passe)
VALUES ('Alice', 'alice@example.com', 'hash_du_mot_de_passe');
```

En PHP :

```php
<?php
$stmt = $pdo->prepare("INSERT INTO utilisateurs (nom, email, mot_passe) VALUES (?, ?, ?)");
$stmt->execute([$nom, $email, $motPasseHache]);
```

### Read - Lire des données

Via phpMyAdmin : onglet **Afficher** pour voir les enregistrements, ou onglet **SQL** pour une requête personnalisée.

En SQL :

```sql
-- Tous les utilisateurs
SELECT * FROM utilisateurs;

-- Un utilisateur par email
SELECT * FROM utilisateurs WHERE email = 'alice@example.com';
```

En PHP :

```php
<?php
$stmt = $pdo->prepare("SELECT * FROM utilisateurs WHERE id = ?");
$stmt->execute([$id]);
$utilisateur = $stmt->fetch(PDO::FETCH_ASSOC);
```

### Update - Modifier des données

Via phpMyAdmin : cliquer sur l'icone de modification d'une ligne, puis valider.

En SQL :

```sql
UPDATE utilisateurs
SET nom = 'Alice Dupont'
WHERE id = 1;
```

En PHP :

```php
<?php
$stmt = $pdo->prepare("UPDATE utilisateurs SET nom = ? WHERE id = ?");
$stmt->execute([$nouveauNom, $id]);
```

### Delete - Supprimer des données

Via phpMyAdmin : cliquer sur **Supprimer** sur la ligne concernée.

En SQL :

```sql
DELETE FROM utilisateurs WHERE id = 1;
```

En PHP :

```php
<?php
$stmt = $pdo->prepare("DELETE FROM utilisateurs WHERE id = ?");
$stmt->execute([$id]);
```

---

## Hashage du mot de passe

Les mots de passe ne doivent jamais être stockés en clair dans la base de données. PHP fournit des fonctions natives pour les hasher de façon sécurisée.



### Hasher un mot de passe à l'inscription

```php
<?php
$motPasse = $_POST['mot_passe'];

// Hashage avec bcrypt (algorithme recommandé)
$motPasseHache = password_hash($motPasse, PASSWORD_BCRYPT);

// Insertion en base
$stmt = $pdo->prepare("INSERT INTO utilisateurs (nom, email, mot_passe) VALUES (?, ?, ?)");
$stmt->execute([$nom, $email, $motPasseHache]);
```

Le hash généré ressemble à ceci (60 caractères) :

```
$2y$10$e0NRq7J3y9Lk8mXvPQs1uOKjT5dHgWbYcZnIaFpMqRtSxUvEwD4Gi
```

### Vérifier un mot de passe à la connexion

```php
<?php
$motPasseSaisi = $_POST['mot_passe'];

// Récupération de l'utilisateur
$stmt = $pdo->prepare("SELECT * FROM utilisateurs WHERE email = ?");
$stmt->execute([$email]);
$utilisateur = $stmt->fetch(PDO::FETCH_ASSOC);

// Comparaison avec le hash stocké
if ($utilisateur && password_verify($motPasseSaisi, $utilisateur['mot_passe'])) {
    echo "Connexion reussie";
} else {
    echo "Email ou mot de passe incorrect";
}
```

### Mise a jour du hash si necessaire

`password_needs_rehash()` permet de détecter si un hash doit être mis à jour (après un changement d'algorithme ou de cout) :

```php
<?php
if (password_needs_rehash($utilisateur['mot_passe'], PASSWORD_BCRYPT)) {
    $nouveauHash = password_hash($motPasseSaisi, PASSWORD_BCRYPT);
    $stmt = $pdo->prepare("UPDATE utilisateurs SET mot_passe = ? WHERE id = ?");
    $stmt->execute([$nouveauHash, $utilisateur['id']]);
}
```


