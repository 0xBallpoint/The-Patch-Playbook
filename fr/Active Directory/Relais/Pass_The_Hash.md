# Protection contre l'Attaque Pass The Hash (PtH)

## Qu'est-ce que l'attaque Pass The Hash ?

L'attaque **Pass The Hash (PtH)** est une technique où un attaquant vole un hash (empreinte) de mot de passe et l'utilise pour s'authentifier à un service sans avoir besoin de connaître le mot de passe en clair. Elle cible principalement les environnements Windows, notamment Active Directory, et repose sur l'exploitation de mécanismes d'authentification faibles, en particulier les mots de passe NTLM (New Technology LAN Manager).

## Comment se protéger contre les attaques Pass The Hash ?

Pour protéger votre infrastructure Active Directory contre les attaques Pass The Hash, il est crucial de suivre plusieurs bonnes pratiques de sécurité. Voici les étapes et configurations recommandées pour limiter les risques :

### 1. **Éviter l'utilisation de NTLM**

NTLM est une méthode d'authentification plus ancienne et plus vulnérable que Kerberos. Si possible, il est recommandé de limiter ou de désactiver NTLM dans votre environnement.

#### Étapes pour désactiver NTLM :
1. **Configurer les GPO pour refuser NTLM** :
   - Ouvrez la console de gestion des stratégies de groupe (GPO).
   - Allez dans : `Configuration Ordinateur > Paramètres Windows > Paramètres de Sécurité > Options de Sécurité`.
   - Trouvez l'option **Réseau : Restreindre l'utilisation des types d'authentification NTLM** et sélectionnez **Refuser NTLM**.
   
2. **Surveiller l'usage de NTLM** avant de le désactiver totalement :
   - Activez la journalisation pour identifier les systèmes et applications qui utilisent NTLM.
   - Suivez les événements d'audit NTLM dans le journal des événements (Event Viewer) via l'ID d'événement 4624.

### 2. **Implémenter l'Authentification Kerberos**

Kerberos est un protocole d'authentification plus sécurisé que NTLM, et il est recommandé de l'utiliser dans les environnements Active Directory.

#### Configurations Kerberos :
- Vérifiez que tous les services prennent en charge Kerberos.
- Utilisez des tickets Kerberos pour l'authentification.
- Surveillez l'utilisation de Kerberos avec l'ID d'événement 4768 (Demande de Ticket TGT).

### 3. **Minimiser les Privilèges des Comptes**

Limiter les privilèges des utilisateurs et la portée des comptes administratifs est essentiel pour réduire les risques d'attaques Pass The Hash.

#### Bonnes pratiques :
- Implémentez le principe du moindre privilège (Least Privilege).
- Utilisez des **comptes d'administration limités** uniquement pour les tâches administratives.
- Séparez les comptes d'administration des comptes d'utilisateurs standards.
- Utilisez **Protected Users** dans Active Directory pour renforcer la sécurité des comptes sensibles.

### 4. **Utiliser des Serveurs de rebond pour l'Administration**

Un serveur "jump" ou de rebond, est un serveur sécurisé utilisé pour se connecter aux systèmes administratifs. En limitant les machines d'administration, vous réduisez la surface d'attaque.

#### Configuration :
- Configurez des **jump servers** pour les connexions administratives.
- Limitez les privilèges des administrateurs sur les machines standards.
- Utilisez des outils comme **Remote Desktop Gateway** pour restreindre l'accès à ces serveurs.

### 5. **Isoler les Identifiants Sensibles**

Protégez les informations d'identification des comptes sensibles en évitant qu'elles ne soient stockées ou mises en cache sur des machines non sécurisées.

#### Étapes :
- Activez **Credential Guard** sur les systèmes Windows pour protéger les hachages NTLM.
- Empêchez les administrateurs locaux d'accéder à distance aux machines utilisateurs.
- Désactivez la mise en cache des informations d'identification à l'aide de la GPO : `Configuration Ordinateur > Stratégies > Modèles d'administration > Système > Gestion des informations d'identification > Ne pas autoriser la mise en cache des informations d’identification`.

### 6. **Surveiller et Auditer les Accès**

La surveillance proactive des accès et des logs peut vous aider à détecter des tentatives de Pass The Hash. Configurez l'audit et la journalisation dans Active Directory.

#### Bonnes pratiques de surveillance :
- Surveillez les événements de connexion suspects via l'ID d'événement 4624 (connexion réussie) et 4625 (connexion échouée).
- Activez l'audit des **logons réseau** pour identifier les connexions qui utilisent des hachages.
- Utilisez des solutions SIEM pour centraliser et analyser les journaux de sécurité.

---

## Références

- [Guide Microsoft : Mitigation des attaques Pass The Hash](https://docs.microsoft.com/fr-fr/windows-server/security/kerberos/kerberos-authentication-protocol)
- [OWASP : Attaque Pass The Hash](https://owasp.org/www-community/attacks/Pass_The_Hash)
