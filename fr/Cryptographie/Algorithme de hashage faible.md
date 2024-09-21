# Risques liés à l'utilisation d'un algorithme de hashage faible (MD5, SHA-1)

## Qu'est-ce qu'un algorithme de hashage faible ?

Les algorithmes de hashage comme **MD5** et **SHA-1** sont considérés comme vulnérables, en particulier aux attaques par collision et par force brute. Leur utilisation dans des contextes sensibles (par exemple, pour le stockage des mots de passe) n'est plus recommandée.
- Attaques par collision : Un attaquant peut créer deux fichiers ou messages différents qui produisent le même hash. Cela permet, par exemple, de falsifier des documents ou de générer des certificats numériques malveillants.
- Force brute et attaques par dictionnaire : Ces algorithmes sont également vulnérables aux attaques par force brute, surtout lorsqu'ils sont utilisés pour hasher des mots de passe sans sel (salt).
- Détournement d'intégrité : Utiliser MD5 ou SHA-1 pour vérifier l'intégrité des fichiers peut être dangereux, car un fichier corrompu ou modifié peut avoir le même hash qu'un fichier sain.

## Comment se protéger ?

Il est recommandé d'utiliser des algorithmes de chiffrements tels que **Argon2id**, **bcrypt** et **PBKDF2** :

### 1. **Implémenter bcrypt**

**bcrypt** est un algorithme de hashage spécialement conçu pour sécuriser les mots de passe. Il inclut un sel intégré et est adaptatif, ce qui le rend résistant aux attaques par force brute.

#### Implémentation en PHP

Utilisez les fonctions natives de PHP pour hasher et vérifier les mots de passe avec bcrypt.

```php
// Hasher un mot de passe avec bcrypt
$mot_de_passe = 'monMotDePasse';
$hash = password_hash($mot_de_passe, PASSWORD_BCRYPT);

// Vérifier un mot de passe hashé
if (password_verify('monMotDePasse', $hash)) {
    echo 'Mot de passe correct';
} else {
    echo 'Mot de passe incorrect';
}
```

#### Implémentation en Python

En Python, utilisez la bibliothèque **bcrypt**.

```python
import bcrypt

# Hasher un mot de passe
mot_de_passe = b"monMotDePasse"
hash_mdp = bcrypt.hashpw(mot_de_passe, bcrypt.gensalt())

# Vérifier un mot de passe
if bcrypt.checkpw(mot_de_passe, hash_mdp):
    print("Mot de passe correct")
else:
    print("Mot de passe incorrect")
```

#### Implémentation en React (via Node.js avec bcryptjs)

React est une bibliothèque frontend, donc vous devrez traiter le hashage côté serveur avec Node.js.

```js
const bcrypt = require('bcryptjs');

// Hasher un mot de passe
const motDePasse = 'monMotDePasse';
const saltRounds = 10;
bcrypt.hash(motDePasse, saltRounds, (err, hash) => {
  if (err) throw err;
  console.log('Mot de passe hashé:', hash);
});

// Vérifier un mot de passe
const hash = "$2a$10$...";
bcrypt.compare(motDePasse, hash, (err, result) => {
  if (result) {
    console.log('Mot de passe correct');
  } else {
    console.log('Mot de passe incorrect');
  }
});
```

### 2. **Implémenter PBKDF2**

**PBKDF2** est un autre algorithme de dérivation de clé sécurisé, recommandé pour le hashage des mots de passe. Il est conçu pour être résistant aux attaques par force brute en rendant l'opération de calcul coûteuse en termes de temps.

#### Implémentation en PHP

Utilisez la fonction native `hash_pbkdf2` de PHP.

```php
$mot_de_passe = 'monMotDePasse';
$sel = random_bytes(16);  // Générer un sel sécurisé
$hash = hash_pbkdf2("sha256", $mot_de_passe, $sel, 10000, 64);
```

#### Implémentation en Python

La bibliothèque standard de Python offre un support pour PBKDF2 via la fonction **hashlib**.

```python
import hashlib
import os

mot_de_passe = b"monMotDePasse"
sel = os.urandom(16)  # Génère un sel sécurisé
hash_mdp = hashlib.pbkdf2_hmac('sha256', mot_de_passe, sel, 100000)
```

#### Implémentation en React (via Node.js avec crypto)

```js
const crypto = require('crypto');

const motDePasse = 'monMotDePasse';
const sel = crypto.randomBytes(16).toString('hex');
crypto.pbkdf2(motDePasse, sel, 600000, 64, 'sha256', (err, derivedKey) => {
  if (err) throw err;
  console.log('Mot de passe hashé:', derivedKey.toString('hex'));
});
```


## **Example de migration d'un site utilisant MD5 vers bcrypt**

Migrer un site web qui stocke actuellement les mots de passe avec **MD5** (ou un autre algorithme faible) vers **bcrypt** nécessite plusieurs étapes pour assurer la compatibilité et la sécurité sans forcer tous les utilisateurs à changer leurs mots de passe immédiatement.

#### Étape 1 : Auditer la base de données

- **Vérifier** où et comment les mots de passe sont stockés. Recherchez les hashs MD5, SHA-1 ou tout autre algorithme faible.
- Identifier s'il y a des "sels" (salts) et leur méthode de génération.

#### Étape 2 : Préparer la transition

- **Ajouter un champ supplémentaire dans la base de données** pour marquer quels mots de passe ont été migrés vers bcrypt.
- Exemple : ajouter une colonne `hash_algorithme` pour indiquer si le mot de passe est en MD5 ou bcrypt.

#### Étape 3 : Mettre à jour le système d'authentification

Mettez à jour votre code d'authentification pour gérer la migration dynamique des mots de passe.

1. Lorsque l'utilisateur se connecte avec son mot de passe MD5 :
   - **Vérifier le mot de passe** en utilisant la méthode MD5 actuelle.
   - Si l'authentification réussit, **rehasher** le mot de passe avec bcrypt et stocker le nouveau hash.
   - Mettre à jour la colonne `hash_algorithme` pour indiquer que le mot de passe utilise bcrypt.

2. Pour les nouveaux utilisateurs :
   - **Hasher les nouveaux mots de passe** avec bcrypt directement lors de la création du compte ou de la modification du mot de passe.

#### Exemple en PHP pour migrer de MD5 à bcrypt :

```php
$mot_de_passe = $_POST['mot_de_passe'];
$hash_enregistre = $utilisateur['hash_mot_de_passe'];
$algorithme = $utilisateur['hash_algorithme'];

// Vérifier si l'utilisateur utilise MD5
if ($algorithme === 'MD5') {
    // Vérifier le mot de passe en utilisant MD5
    if (md5($mot_de_passe) === $hash_enregistre) {
        // Authentification réussie, rehasher avec bcrypt
        $nouveau_hash = password_hash($mot_de_passe, PASSWORD_BCRYPT);
        // Mettre à jour dans la base de données
        $requete = "UPDATE utilisateurs SET hash_mot_de_passe = ?, hash_algorithme = 'bcrypt' WHERE id = ?";
        $stmt = $db->prepare($requete);
        $stmt->execute([$nouveau_hash, $utilisateur['id']]);
    } else {
        echo "Mot de passe incorrect.";
    }
} elseif (password_verify($mot_de_passe, $hash_enregistre)) {
    // Le mot de passe est déjà en bcrypt
    echo "Connexion réussie.";
} else {
    echo "Mot de passe incorrect.";
}
```

#### Étape 4 : Forcer la mise à jour des mots de passe (optionnelle)

Dans certains cas, il peut être pertinent de forcer les utilisateurs à **changer leur mot de passe** pour garantir que tous les mots de passe sont migrés.

- Envoyez des notifications pour informer les utilisateurs de la nécessité de mettre à jour leur mot de passe.
- Assurez-vous que le nouveau mot de passe est hasher avec bcrypt ou PBKDF2.

#### Étape 5 : Supprimer les anciens algorithmes (MD5, SHA-1)

Une fois que la majorité des utilisateurs ont migré leurs mots de passe vers bcrypt, vous pouvez **désactiver le support pour MD5**.

- Effectuez un audit pour vous assurer que tous les utilisateurs restants ont migré.
- Supprimez le code d'authentification basé sur MD5.

#### Étape 6 : Mise à jour et vérification

- **Surveillez les journaux** pour identifier les éventuels problèmes d'authentification pendant la migration.
- Testez en profondeur votre système d'authentification pour vous assurer que la migration n'a pas introduit de nouvelles vulnérabilités.

---

## Références

- [OWASP : Stockage sécurisé des mots de passe](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [PHP : password_hash](https://www.php.net/manual/fr/function.password-hash.php)
- [Python : hashlib](https://docs.python.org/3/library/hashlib.html)
- [NIST : Recommandations pour le stockage des mots de passe](https://pages.nist.gov/800-63-3/sp800-63b.html)
