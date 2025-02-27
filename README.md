# Atelier : GPO

## Préparation des VM

Tu as besoin de **3 VM** sous VirtualBox :

- **1 VM serveur** : Windows Server 2022  
- **2 VM clients** : Windows 10  

Chaque VM doit avoir **une carte réseau** en mode **Réseau interne** avec le même nom de réseau.

## Configuration des VM

### VM Clients  
Chaque machine :  
- Est sur le domaine **lab.lan**  
- A déjà eu une session d'ouverte avec les comptes du domaine **User1** et **User2**  

### VM Serveur  
- Les rôles **AD-DS** et **DNS** sont installés  
- Un domaine **lab.lan** est installé  
- Les **OU suivantes** sont créées à la racine du domaine :  
  - **LabSecurite**  
  - **LabOrdinateurs**  
  - **LabUtilisateurs**  

### Groupes créés dans l’OU *LabSecurite* :  
- **GrpComputers7Zip**  
- **GrpComputersFirefox**  
- **GrpUsersChrome**  
- **GrpUsersWallpaper-Green**  
- **GrpUsersWindowsRestrictions**  

### Utilisateurs créés dans l’OU *LabUtilisateurs* :  
- **User1** → Membre du groupe **GrpUsersChrome**  
- **User2** → Membre des groupes **GrpUsersWindowsRestrictions** et **GrpUsersWallpaper-Green**  

### Objets ordinateurs dans l’OU *LabOrdinateurs* :  
- **CLIENT1** → Membre du groupe **GrpComputers7Zip**  
- **CLIENT2** → Membre du groupe **GrpComputersFirefox**  

### Partage de fichiers  
- Un dossier partagé **Ressources** est créé à la racine du système de fichiers  
- Ce dossier est partagé sur le réseau sous le même nom **Ressources**  
- Il est accessible par le groupe **Everyone** avec les permissions **Read**  

# Détails de la configuration des VM  

| Fonction de la VM  | Serveur          | Client               |
|--------------------|-----------------|----------------------|
| **Nom**           | WINSERV1         | CLIENT1 / CLIENT2   |
| **OS**            | Windows Server 2022 | Windows 10      |
| **OS version**    | Standard Desktop Experience | Professionnel |
| **RAM**           | 4/8 Go           | 2/4 Go               |
| **Langue à installer** | English (US) | French              |
| **Time and currency / keyboard** | French | French        |
| **Carte réseau VirtualBox** | Réseau privé | Réseau privé |
| **Adresse IP**    | 10.10.5.10/24    | 10.10.5.100/24 et 10.10.5.200/24 |
| **Passerelle**    | -                | -                    |
| **DNS**           | 10.10.5.10 ou 127.0.0.1 | 10.10.5.10 |
| **Utilisateur local** | Administrator | Wilder          |
| **Firewall**      | Désactivé        | Désactivé             |

