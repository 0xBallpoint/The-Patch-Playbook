# Le protocol SMB

Le protocole Server Block Message (SMB) est utilisé pour le partage de fichiers sur les systèmes Windows. Dans le cas ou la signature SMB n'est pas activé, vous courez le risque que votre trafic SMB soit intercepté par un acteur malveillant (homme du milieu). Cela signifie qu'un attaquant interne peut intercepter les sessions de partage actives sur votre réseau.

## La signature SMB, C'est quoi ?

Le protocole SMB signe chaque paquet avec une signature numérique afin que le client et le serveur puissent confirmer leur origine ainsi que l'authenticité de la communication. Lorsque la signature SMB est activée, si un attaquant tente de voler une session SMB, il lui sera impossible de modifier les paquets, l'empêchant ainsi d'intercepter et de rejouer la requête SMB.
Il est important de noter que la signature sert à confirmer l'authenticité de l'émetteur et du destinataire, ce n'est donc pas un chiffrement. Le chiffrement SMB a été ajouté dans la version SMB 3.0 et peut être utile dans le cas ou vous partager des informations confidentielles ou sensibles sur le réseau.

## Activation de la signature SMB (Stratégie de groupe)

### **1. Ouverture de la Gestion de la stratégie de groupe**

Utilisez le Gestionnaire de serveur et accédez à Outils > Gestion de la stratégie de groupe.

Ou exécutez 'gpmc.msc' dans PowerShell ou l'Invite de commandes.

### **2. Création ou modification de la stratégie**

Créez une nouvelle stratégie si nécessaire, ou éditez une stratégie existante en fonction de vos besoins.

### **3. Navigation vers la section appropriée**

Dans la stratégie, allez à Configuration de l'ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Options de sécurité.

### **4. Modification des éléments de stratégie**

Il y a 4 éléments de stratégie que vous pouvez modifier en fonction de vos besoins.
- Activez l’élément nommé Client réseau Microsoft : Signer numériquement les communications (toujours)
- Activez l’élément nommé Client réseau Microsoft : Signer numériquement les communications (Si les clients acceptent)
- Activez l’élément nommé Serveur réseau Microsoft : Signer numériquement les communications (toujours)
- Activez l’élément nommé Serveur réseau Microsoft : Signer numériquement les communications (Si les clients acceptent)

#### Activez l’élément nommé Client réseau Microsoft : Signer numériquement les communications (toujours)

L'activation de cette stratégie garantit que le client SMB exigera toujours la signature des paquets SMB. Si le serveur refuse de prendre en charge la signature des paquets SMB avec le client, ce dernier ne communiquera pas avec le serveur. Par défaut, cette stratégie est désactivée, ce qui signifie que SMB est autorisé par défaut sans exiger la signature des paquets. Il reste possible de négocier la signature des paquets, mais cela n'est pas nécessaire pour fonctionner.
Si vous activez cette stratégie de groupe, SMB sera toujours signé numériquement. Autrement dit, si la machine Windows tente de se connecter à un serveur SMB qui ne prend pas en charge la signature des paquets SMB, la connexion échouera.

#### Activez l’élément nommé Client réseau Microsoft : Signer numériquement les communications (Si les clients acceptent)

Cette stratégie est activée par défaut et détermine si le client SMB tente de négocier la signature des paquets SMB avec le serveur. Si elle est désactivée, le client ne demandera pas la signature des paquets SMB.

#### Activez l’élément nommé Serveur réseau Microsoft : Signer numériquement les communications (toujours)

Cette option de stratégie contrôle si le serveur fournissant SMB requiert la signature des paquets. Elle détermine si la signature des paquets SMB doit être négociée avant toute communication supplémentaire avec un client SMB.

Par défaut, ce paramètre est activé pour les contrôleurs de domaine, mais désactivé pour les autres serveurs membres du domaine. Activer cette option nécessitera une communication signée numériquement via SMB, ce qui peut interrompre les connexions SMB dans le cas ou le client ne prend pas en charge la signature SMB.

####  Activez l’élément nommé Serveur réseau Microsoft : Signer numériquement les communications (Si les clients acceptent)

Cette option de stratégie détermine si le serveur SMB négociera la signature des paquets SMB avec les clients qui en font la demande. Lorsque ce paramètre est activé, le serveur SMB négociera la signature des paquets SMB selon la demande du client. Si la signature des paquets SMB est activée côté client, elle sera négociée par le serveur. Par défaut, cette stratégie est uniquement activée sur les contrôleurs de domaine.

En suivant ces étapes, vous pourrez configurer la signature SMB de manière précise via la stratégie de groupe.
