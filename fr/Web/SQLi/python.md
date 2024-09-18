# Correction des Injections SQL (SQLi) en Python

L'injection SQL (SQLi) est une vulnérabilité critique qui permet à un attaquant d'exécuter des commandes SQL non autorisées sur une base de données en manipulant les entrées utilisateur. Ce document détaille les meilleures pratiques pour sécuriser une application Python contre les attaques par injection SQL.

## Solution : Utilisation de Requêtes Paramétrées avec `sqlite3` ou `MySQLdb`

Pour éviter les injections SQL en Python, la méthode recommandée est d'utiliser des **requêtes paramétrées**, qui séparent les données fournies par l'utilisateur des commandes SQL.

### Étapes pour Corriger l'Injection SQL avec `sqlite3` ou `MySQLdb`

### 1. Connexion à la Base de Données

Voici un exemple de connexion à une base de données SQLite en Python, qui peut être adapté pour d'autres bibliothèques comme `MySQLdb` ou `psycopg2` (pour PostgreSQL).

#### Exemple avec `sqlite3` :
```python
import sqlite3

try:
    conn = sqlite3.connect('ma_base_de_donnees.db')
    cursor = conn.cursor()
except sqlite3.Error as e:
    print(f"Erreur de connexion : {e}")
```

#### Exemple avec `MySQLdb` :
```python
import MySQLdb

try:
    conn = MySQLdb.connect(host="localhost", user="utilisateur", passwd="mot_de_passe", db="ma_base_de_donnees")
    cursor = conn.cursor()
except MySQLdb.Error as e:
    print(f"Erreur de connexion : {e}")
```

### 2. Utilisation de Requêtes Paramétrées

Les requêtes paramétrées sont essentielles pour éviter les injections SQL. Elles permettent de lier les données utilisateur aux requêtes sans que ces données ne soient interprétées comme du code SQL.

#### Exemple incorrect (vulnérable à l'injection SQL) :
```python
id = input("Entrez l'ID de l'utilisateur : ")
query = f"SELECT * FROM utilisateurs WHERE id = {id}"  # Ne pas faire ceci !
cursor.execute(query)
```

#### Exemple correct (sécurisé avec une requête paramétrée) :
```python
id = input("Entrez l'ID de l'utilisateur : ")
query = "SELECT * FROM utilisateurs WHERE id = ?"
cursor.execute(query, (id,))
result = cursor.fetchall()
```

Dans cet exemple, `?` est un **paramètre lié**, et les données utilisateur (`id`) sont échappées automatiquement par la bibliothèque, empêchant ainsi les injections SQL.

### 3. Requêtes Paramétrées pour `MySQLdb` (MySQL)

Pour `MySQLdb`, les paramètres liés utilisent `%s` comme marqueur dans la requête.

#### Exemple avec `MySQLdb` :
```python
id = input("Entrez l'ID de l'utilisateur : ")
query = "SELECT * FROM utilisateurs WHERE id = %s"
cursor.execute(query, (id,))
result = cursor.fetchall()
```

### 4. Prévenir les Injections dans les INSERT et UPDATE

Les requêtes paramétrées fonctionnent également pour les requêtes **INSERT** et **UPDATE**.

#### Exemple pour un INSERT sécurisé avec `sqlite3` :
```python
nom = input("Entrez le nom : ")
email = input("Entrez l'email : ")
query = "INSERT INTO utilisateurs (nom, email) VALUES (?, ?)"
cursor.execute(query, (nom, email))
conn.commit()
```

#### Exemple pour un UPDATE sécurisé avec `MySQLdb` :
```python
id = input("Entrez l'ID : ")
nom = input("Entrez le nouveau nom : ")
query = "UPDATE utilisateurs SET nom = %s WHERE id = %s"
cursor.execute(query, (nom, id))
conn.commit()
```

### 5. Utilisation des Filtrages et Validations

Même si les requêtes paramétrées sont une excellente défense contre les injections SQL, il est toujours recommandé de **valider et filtrer** les données utilisateur avant de les utiliser.

#### Exemple de validation :
```python
id = input("Entrez l'ID : ")
if not id.isdigit():
    print("ID invalide")
    exit()
```

### Autres Bonnes Pratiques :
- **Limiter les privilèges des utilisateurs de la base de données** : L'utilisateur de la base de données utilisé par l'application Python ne devrait avoir que les privilèges strictement nécessaires (lecture, écriture, mais pas d'accès administrateur).
- **Journaliser les erreurs et tentatives d'accès** : Mettre en place des journaux pour suivre les requêtes échouées et détecter des tentatives d'injection.
- **Mettre à jour les bibliothèques** : Assurez-vous que Python et les bibliothèques de gestion de base de données (comme `sqlite3`, `MySQLdb` ou `psycopg2`) sont à jour pour bénéficier des derniers correctifs de sécurité.

---

## Références

- [Documentation Python : sqlite3](https://docs.python.org/3/library/sqlite3.html)
- [OWASP : SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
