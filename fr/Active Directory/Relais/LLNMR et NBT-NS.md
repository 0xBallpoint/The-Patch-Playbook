# Protection contre l'exploitation des protocoles LLMNR/NBT-NS

## Les protocoles LLMNR et NBT-NS ?

L'attaque via **Responder** exploite les protocoles LLMNR (Link-Local Multicast Name Resolution) et NBT-NS (NetBIOS Name Service), qui sont activés par défaut dans les environnements Windows pour résoudre les noms d'hôtes non trouvés via DNS. Un attaquant peut utiliser l'outil Responder pour intercepter et répondre aux requêtes de résolution de noms, capturer les informations d'identification (hashs de mots de passe NTLM), et tenter des attaques de type "Pass The Hash" ou déchiffrer les hashs pour obtenir les mots de passe en clair.

## Comment se protéger contre l'attaque Responder ?

Pour éviter cette attaque, il est crucial de désactiver les protocoles vulnérables, renforcer la sécurité des réponses aux requêtes réseau, et surveiller les activités suspectes.

### 1. **Désactiver LLMNR et NBT-NS**

La première étape pour protéger votre infrastructure contre les attaques Responder est de désactiver les protocoles LLMNR et NBT-NS, qui sont rarement nécessaires dans les environnements modernes.

#### Désactivation de LLMNR via GPO :
1. Ouvrez la **console de gestion des stratégies de groupe (GPO)**.
2. Allez dans : `Configuration Ordinateur > Modèles d'administration > Réseau > Protocole de résolution de noms multi-diffusion LLMNR`.
3. Activez l'option **Désactiver le protocole LLMNR**.

#### Désactivation de NBT-NS sur les interfaces réseau :
1. Ouvrez les **Paramètres réseau** de chaque machine (ou via GPO).
2. Dans les propriétés TCP/IP de chaque carte réseau :
   - Cliquez sur **Avancé** > **WINS** > **Désactiver NetBIOS sur TCP/IP**.
3. Appliquez cette configuration à toutes les machines du domaine.

### 2. **Mettre en œuvre SMB Signing (Signature SMB)**

Le **SMB Signing** permet de signer numériquement les communications SMB, empêchant un attaquant de modifier les paquets SMB capturés par Responder.

#### Activer SMB Signing via GPO :
1. Ouvrez la **console GPO**.
2. Allez dans : `Configuration Ordinateur > Paramètres Windows > Paramètres de Sécurité > Stratégies Locales > Options de Sécurité`.
3. Trouvez l'option **Microsoft network client : Digitaly sign communications (always)** et activez-la.
4. Appliquez également cette règle au serveur avec l'option **Microsoft network server : Digitaly sign communications (always)**.

### 3. **Segmentation du Réseau et Isolation**

Une bonne segmentation du réseau peut limiter la portée des attaques Responder en empêchant les attaquants de capturer les requêtes LLMNR/NBT-NS sur les réseaux critiques.

#### Bonnes pratiques de segmentation :
- Segmentez les réseaux utilisateurs des réseaux administratifs ou des systèmes sensibles.
- Utilisez des VLANs pour isoler les services critiques.
- Restreignez les communications entre les différents segments du réseau.

### 4. **Auditer et Surveiller les Protocoles Réseau**

La surveillance des événements réseau liés aux protocoles LLMNR et NBT-NS peut vous alerter en cas d'attaque Responder en cours. Configurez la journalisation pour capturer ces événements et les analyser.

#### Surveiller les événements liés à LLMNR/NBT-NS :
- Utilisez des solutions SIEM (Security Information and Event Management) pour centraliser les logs.
- Recherchez des comportements anormaux, tels que des réponses LLMNR ou NBT-NS provenant de machines non autorisées.
- Activez l'audit de résolution de noms sur les machines pour capturer les requêtes non résolues.

---

## Références

- [Microsoft : Sécuriser les communications réseau](https://docs.microsoft.com/fr-fr/windows/security/threat-protection/security-policy-settings/microsoft-network-client-digitally-sign-communications-always)
- [OWASP : Attaques LLMNR/NBT-NS](https://owasp.org/www-community/attacks/LLMNR-NBT-NS-poisoning)
