# Correction des Injections SQL (SQLi) en PHP

L'injection SQL (SQLi) est une vulnérabilité critique qui permet à un attaquant d'exécuter des commandes SQL arbitraires sur une base de données en modifiant les entrées utilisateur non sécurisées. Ce document détaille les meilleures pratiques pour protéger une application PHP contre les attaques par injection SQL.

## Solution : Utilisation de Requêtes Préparées avec PDO

La méthode recommandée pour éviter les injections SQL en PHP est d'utiliser **PDO (PHP Data Objects)** avec des requêtes préparées, qui permettent de séparer la requête SQL des données fournies par l'utilisateur.

### Étapes pour Corriger l'Injection SQL avec PDO

### 1. Connexion à la base de données avec PDO

Avant d'utiliser des requêtes préparées, il est important de bien configurer la connexion à la base de données avec **PDO**. Voici un exemple pour se connecter à une base de données MySQL :

```php
<?php
try {
    $dsn = 'mysql:host=localhost;dbname=ma_base_de_donnees';
    $username = 'utilisateur';
    $password = 'mot_de_passe';
    $options = [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,  // Active les exceptions pour les erreurs
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC // Mode de récupération des résultats
    ];
    
    $pdo = new PDO($dsn, $username, $password, $options);
} catch (PDOException $e) {
    die('Erreur de connexion : ' . $e->getMessage());
}
```

### 2. Utilisation de Requêtes Préparées pour Éviter les Injections SQL

Lorsque vous effectuez une requête SQL en PHP, n'insérez jamais directement les données utilisateurs dans la requête. Utilisez des **requêtes préparées** avec des **paramètres liés** pour garantir que les données ne seront pas interprétées comme du code SQL.

#### Exemple incorrect (vulnérable à l'injection SQL) :
```php
<?php
$id = $_GET['id'];
$sql = "SELECT * FROM utilisateurs WHERE id = $id";  // Ne pas faire ceci !
$result = $pdo->query($sql);
```

#### Exemple correct (sécurisé avec une requête préparée) :
```php
<?php
$id = $_GET['id'];
$sql = "SELECT * FROM utilisateurs WHERE id = :id";
$stmt = $pdo->prepare($sql);
$stmt->execute(['id' => $id]);
$result = $stmt->fetchAll();
```

Dans cet exemple, `:id` est un **paramètre lié**, et les données utilisateur (`$id`) sont automatiquement échappées avant d'être intégrées à la requête SQL, empêchant toute injection.

### 3. Utilisation des Paramètres Liés pour Différents Types de Données

Les requêtes préparées avec PDO permettent également de spécifier explicitement le type de données pour chaque paramètre, offrant ainsi une couche de sécurité supplémentaire.

```php
<?php
$id = $_GET['id'];
$sql = "SELECT * FROM utilisateurs WHERE id = :id";
$stmt = $pdo->prepare($sql);
$stmt->bindParam(':id', $id, PDO::PARAM_INT);  // Assure que $id est un entier
$stmt->execute();
$result = $stmt->fetchAll();
```

### 4. Prévenir les Injections dans les INSERT et UPDATE

Les requêtes préparées sont également efficaces pour protéger les **INSERT** et **UPDATE**.

#### Exemple pour un INSERT sécurisé :
```php
<?php
$nom = $_POST['nom'];
$email = $_POST['email'];
$sql = "INSERT INTO utilisateurs (nom, email) VALUES (:nom, :email)";
$stmt = $pdo->prepare($sql);
$stmt->execute(['nom' => $nom, 'email' => $email]);
```

#### Exemple pour un UPDATE sécurisé :
```php
<?php
$id = $_POST['id'];
$nom = $_POST['nom'];
$sql = "UPDATE utilisateurs SET nom = :nom WHERE id = :id";
$stmt = $pdo->prepare($sql);
$stmt->execute(['nom' => $nom, 'id' => $id]);
```

### 5. Utilisation des Filtrages et Validations

Bien que les requêtes préparées soient efficaces contre les injections SQL, il est également recommandé de **valider et filtrer les entrées utilisateur** avant leur utilisation dans une requête.

#### Exemple de validation :
```php
<?php
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);
if ($id === false) {
    die('ID invalide');
}
```

### Autres Bonnes Pratiques :
- **Limiter les privilèges des utilisateurs de la base de données** : L'utilisateur de la base de données utilisé par votre application PHP ne doit avoir que les privilèges nécessaires (lecture/écriture, pas d'accès administratif).
- **Journaliser les accès et les erreurs** : Enregistrer les erreurs de connexion et les requêtes échouées pour faciliter le diagnostic des tentatives d'injection.
- **Tenir à jour votre logiciel** : Assurez-vous que PHP, MySQL, et les bibliothèques utilisées sont toujours à jour pour bénéficier des derniers correctifs de sécurité.

---

## Références

- [Documentation PHP : PDO](https://www.php.net/manual/fr/book.pdo.php)
- [OWASP : SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
