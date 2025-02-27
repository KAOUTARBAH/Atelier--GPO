# Atelier : GPO

## Préparation des VM

Tu as besoin de **3 VM** sous VirtualBox :

- **1 VM serveur** : Windows Server 2022  
- **2 VM clients** : Windows 10  

Chaque VM doit avoir **une carte réseau** en mode **Réseau interne** avec le même nom de réseau.

## Configuration des VM

### VM Clients  
Chaque machine :  
- Est sur le domaine **ranka.fr**  
- A déjà eu une session d'ouverte avec les comptes du domaine **User1** et **User2**  

### VM Serveur  
- 1️⃣. Les rôles **AD-DS** et **DNS** sont installés 
    - **Get-WindowsFeature -Name DNS** :Tapez la commande suivante pour vérifier si le rôle DNS est installé
    - **Get-ADForest**: pour afficher les informations relatives à la forêt.
    - **Get-ADDomaineController**: pour afficher les informations relatives au role contrôleur de domaine.
    
![Test dns ad](https://github.com/KAOUTARBAH/Atelier--GPO/blob/main/images/testAD.png)
![Test forest](https://github.com/KAOUTARBAH/Atelier--GPO/blob/main/images/RoleController.png)

- 2️⃣. Un domaine **ranka.fr** est installé  
- Les **OU (Unités Organisationnelles) suivantes** sont créées à la racine du domaine :  
  - **RankaSecurite**  
  - **RankaOrdinateurs**  
  - **RankaUtilisateurs**  

#### Accéder à la racine du domaine :
- Dans l'outil, trouvez votre domaine **ranka.fr** dans la liste à gauche.

#### Créer une nouvelle OU :
- Faites un clic droit sur le domaine **ranka.fr** et sélectionnez **Nouveau > Unité organisationnelle**.
- Dans la fenêtre qui apparaît, donnez un nom à l'OU. Par exemple, pour créer l'OU **RankaSecurite**, tapez **"RankaSecurite"**.
- Cliquez sur **OK** pour valider.

#### Répétez cette procédure pour les autres OU :

- **RankaOrdinateurs**
- **RankaUtilisateurs**

#### Créer l'OU RankaUtilisateurs via PowerShell
    ```bash
    New-ADOrganizationalUnit -Name "RankaUtilisateurs" -Path "DC=ranka,DC=fr"

![ou](https://github.com/KAOUTARBAH/Atelier--GPO/blob/main/images/ou.png)

### 3️⃣. Groupes créés dans l’OU *RankaSecurite* :  
    - **GrpComputers7Zip**  
    - **GrpComputersFirefox**  
    - **GrpUsersChrome**  
    - **GrpUsersWallpaper-Green**  
    - **GrpUsersWindowsRestrictions**  

- Ouvrir Active Directory Users and Computers :
- Allez dans le menu Démarrer, tapez Active Directory Users and Computers, et ouvrez-le.
**Accédez à l'OU RankaSecurite :

- Dans le volet de gauche, trouvez votre domaine ranka.fr, puis ouvrez l'OU RankaSecurite.
Créer un groupe :

- Faites un clic droit sur l'OU RankaSecurite, puis choisissez Nouveau > Groupe.
Dans la fenêtre qui apparaît, donnez un nom au groupe (par exemple, GrpComputers7Zip).
Cliquez sur OK pour créer le groupe.
Répétez l'opération pour les autres groupes :

    - GrpComputersFirefox
    - GrpUsersChrome
    - GrpUsersWallpaper-Green
    - GrpUsersWindowsRestrictions

#### Créer le groupe GrpComputers7Zip dans l'OU RankaSecurite via powershell
    ```bash
    New-ADGroup -Name "GrpComputers7Zip" -Path "OU=RankaSecurite,DC=ranka,DC=fr" -GroupScope Global -PassThru
![groupeSec](https://github.com/KAOUTARBAH/Atelier--GPO/blob/main/images/groupeSec.png)


### 4️⃣. Utilisateurs créés dans l’OU *RankaUtilisateurs* :  
- **User1** → Membre du groupe **GrpUsersChrome**  
- **User2** → Membre des groupes **GrpUsersWindowsRestrictions** et **GrpUsersWallpaper-Green**  

1. Ouvrez **Active Directory Users and Computers**.
2. Accédez à l’OU **RankaUtilisateurs**.
3. Faites un **clic droit** dans l’OU, puis sélectionnez **Nouveau → Utilisateur**.
4. Remplissez les champs suivants :
   - **Nom d’utilisateur** : `User1`
   - **Mot de passe** : définissez un mot de passe sécurisé.
   - **Option** : Cochez **L’utilisateur doit changer de mot de passe à la prochaine ouverture de session** (si nécessaire).
5. Cliquez sur **Suivant**, puis **Terminer**.
6. Répétez ces étapes pour créer `User2`.

#### 1. Création de User1
New-ADUser -Name "User1" -SamAccountName "User2" -UserPrincipalName "User1@ranka.fr" -Path "OU=RankaUtilisateurs,DC=ranka,DC=fr" -AccountPassword (ConvertTo-SecureString "MotDePasse123!" -AsPlainText -Force) -Enabled $true

### 2. Ajout des utilisateurs aux groupes via powershell
1. Dans **Active Directory Users and Computers**, ouvrez l'OU **RankaUtilisateurs**.
2. Faites un **double-clic** sur `User1`, puis allez dans l'onglet **Membre de**.
3. Cliquez sur **Ajouter** et tapez **GrpUsersChrome**, puis validez.
4. Faites de même pour `User2`, mais ajoutez-le aux groupes suivants :
   - `GrpUsersWindowsRestrictions`
   - `GrpUsersWallpaper-Green`
5. Cliquez sur **OK** pour enregistrer.

### Ajouter User1 au groupe GrpUsersChrome via powershell
Add-ADGroupMember -Identity "GrpUsersChrome" -Members "User1"

![users](https://github.com/KAOUTARBAH/Atelier--GPO/blob/main/images/users.png)

### 5️⃣. Objets ordinateurs dans l’OU *RankaOrdinateurs* :  
- **CLIENT1** → Membre du groupe **GrpComputers7Zip**  
- **CLIENT2** → Membre du groupe **GrpComputersFirefox**  

### 6️⃣. Partage de fichiers  
- Un dossier partagé **Ressources** est créé à la racine du système de fichiers  
- Ce dossier est partagé sur le réseau sous le même nom **Ressources**  
- Il est accessible par le groupe **Everyone** avec les permissions **Read**  

## Détails de la configuration des VM  

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


## Faut-il activer la délégation DNS ?

### 🔹 OUI, si :
- Vous utilisez un serveur DNS distinct pour gérer votre domaine (exemple : un autre serveur DNS gère la zone parent).
- Vous avez une infrastructure avec plusieurs domaines et souhaitez déléguer la gestion DNS à un autre serveur.

### 🔹 NON (Ignorer l’avertissement), si :
- Votre serveur Active Directory est aussi votre serveur DNS principal (cas le plus courant).
- Vous installez AD dans un nouveau domaine sans sous-domaines.
