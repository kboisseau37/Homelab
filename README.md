# Homelab — donner une seconde vie à un Acer Aspire E15

Transformer un ancien PC portable en serveur domestique et NAS, tout en apprenant l'administration Linux, le réseau et la sauvegarde. À terme : héberger des photos avec Immich et expérimenter une IA locale utilisable sans Internet.

> **État au 6 octobre 2026 : inventaire matériel et sauvegarde réalisés. Debian n'est pas encore installé.** Ce dépôt est un journal de préparation, puis servira à documenter les opérations au fur et à mesure.

## Sommaire

- [Avancement et sources](#avancement-et-sources)
- [Objectifs et choix de Debian](#objectifs-et-choix-de-debian)
- [Inventaire matériel](#inventaire-matériel)
- [Architecture actuelle et cible](#architecture-actuelle-et-cible)
- [Prérequis](#prérequis)
- [Inventaire avec PowerShell](#inventaire-avec-powershell)
- [Captures et résultats observés](#captures-et-résultats-observés)
- [Sauvegarde des données](#sauvegarde-des-données)
- [Prochaine étape : Debian](#prochaine-étape--debian)
- [Sécurité](#sécurité)
- [Roadmap](#roadmap)
- [Utiliser ce projet](#utiliser-ce-projet)
- [Références](#références)

## Avancement et sources

| Étape | Statut |
|---|---|
| Définir le projet serveur/NAS | Réalisé |
| Choisir Linux et Debian | En cours |
| Identifier le matériel | Réalisé |
| Identifier des données sur C: et D: | Réalisé |
| Copier les documents sur un disque externe | Réalisé |
| Vérifier la copie et débrancher le disque externe | Réalisé |
| Télécharger l'ISO et trouver une clé USB | À faire |
| Créer la clé et installer Debian | À faire |
| Configurer SSH, NAS, Immich | À faire |

Sources : conversation « Serveur maison avec vieux PC », renseignements matériels transmis dans la demande et capture originale récupérée. L'accès à la conversation n'a retourné que ses échanges récents, sans page plus ancienne. Les premières commandes exactes et les autres images ne sont donc pas toutes disponibles. Les exemples reconstitués ci-dessous sont explicitement identifiés.

## Objectifs et choix de Debian

Le serveur doit permettre de partager des fichiers sur le réseau domestique, centraliser certaines données et apprendre à administrer une machine à distance. Un NAS est un stockage accessible par le réseau ; il ne remplace pas une sauvegarde indépendante.

Linux/Debian est le choix retenu pour construire une installation sobre, administrable en ligne de commande. Le plan est d'installer Debian sans environnement de bureau, puis d'accéder au serveur depuis le PC principal avec SSH. Ce choix est acté, mais aucun système Linux opérationnel n'a encore été validé sur cet Acer.

Immich est un futur service de gestion de photos et vidéos. Docker est envisagé pour les services. Leur installation, leurs besoins et leur configuration seront documentés plus tard, selon les versions utilisées et les ressources réellement disponibles.


## Inventaire matériel

| Composant | Matériel relevé | Usage envisagé / observation |
|---|---|---|
| Portable | Acer Aspire E15 E5-553G-T85N | Machine réutilisée pour le homelab |
| Processeur | AMD A10-9600P | Processeur identifié dans l'inventaire fourni |
| Mémoire | 8 Go DDR4, 2 × 4 Go | Deux modules Kingston visibles sur la capture |
| SSD | Kingston A400, 480 Go | Cible prévue pour Debian et les applications |
| HDD | Toshiba MQ01ABD100, 1 To | Stockage de données envisagé ; devenir des partitions à décider |
| Graphique | Radeon R8 M445DX, 2 Go | Information fournie ; compatibilité IA non testée |
| Ethernet | Realtek PCIe GbE Family Controller | Capacité Gigabit identifiée ; câble déconnecté au relevé |
| Wi-Fi | Qualcomm Atheros QCA9377 | Connecté au relevé, vitesse de liaison affichée : 433,3 Mbit/s |

Les capacités commerciales des disques ne sont pas les volumes libres. Leur état de santé et leur disposition exacte devront être contrôlés avant l'installation. Aucune mesure de débit de transfert n'a été réalisée.

## Architecture actuelle et cible

### Situation observée pendant l'inventaire

```text
Acer sous Windows
├── C: : données présentes, associé au SSD dans les échanges
├── D: : données présentes, associé au HDD dans les échanges
├── Wi-Fi : connecté lors du relevé
└── Ethernet : déconnecté lors du relevé

Documents copiés → disque dur externe
```

L'association C:/SSD et D:/HDD vient du compte rendu de la conversation. Elle doit être recontrôlée par modèle et capacité avant tout partitionnement.

### Cible prévue — non déployée

```text
PC principal / appareils du foyer
              │ réseau local
         Box / routeur
              │ câble Ethernet
       Acer / Debian minimal
       ├── SSD 480 Go : système et applications
       ├── HDD 1 To : stockage à organiser
       ├── SSH : administration distante
       ├── Samba : partage de fichiers envisagé
       ├── Docker / Immich : étape ultérieure
       └── IA locale : expérimentation ultérieure

Sauvegarde indépendante → disque externe
```

Ethernet est privilégié pour le serveur. L'inscription « GbE » indique une interface Gigabit ; le débit négocié devra être vérifié une fois le câble branché. Le HDD est un disque unique, sans redondance prévue à ce stade.

## Prérequis

- Le portable Acer et son alimentation, avec une ventilation dégagée.
- Le PC principal pour préparer le support et administrer le serveur.
- Un disque externe contenant les fichiers à conserver, avec vérification de lecture.
- Une clé USB effaçable d'au moins 4 Go, selon le plan initial.
- Un câble Ethernet et un port disponible sur la box ou le routeur.
- Une connexion Internet pour télécharger l'ISO et les paquets de l'installation netinst.
- VS Code pour modifier cette documentation ; Git et un compte GitHub pour une publication ultérieure.

## Inventaire avec PowerShell

Ces commandes concernent **le diagnostic Windows de l'Acer**, pas le PC principal. Elles ne formatent aucun disque. Elles ne sont pas exécutées par ce projet de documentation.

### Commandes attestées par la capture

```powershell
Get-CimInstance Win32_PhysicalMemory | Format-Table DeviceLocator, Manufacturer, Capacity, Speed, PartNumber
Get-NetAdapter
```

La première interroge les modules mémoire, puis présente certaines propriétés sous forme de tableau. La seconde affiche les interfaces réseau et leur état au moment du relevé.

### Comprendre le pipe `|`

Le caractère `|` transmet les objets produits par la commande de gauche à la commande de droite. PowerShell transmet des objets avec des propriétés, pas simplement les caractères d'un écran.

```text
Get-CimInstance Win32_PhysicalMemory
      objets représentant les modules mémoire
                         │ pipe |
                         ▼
Format-Table DeviceLocator, Manufacturer, Capacity, Speed, PartNumber
      affichage de propriétés choisies
```

Les virgules séparent les propriétés à afficher. Sur la capture, un essai sépare `PartNumber` en deux mots, ce qui produit une erreur de paramètre positionnel. La commande corrigée utilise `PartNumber` en un seul mot et affiche les deux modules. Un autre essai transmet `device .\locale.nls` à `Format-Table` et échoue : ce texte ne correspond pas à la liste de propriétés souhaitée.

`Format-Table` sert à présenter les résultats. Pour sélectionner des propriétés destinées à un traitement ou à un export, utiliser plutôt `Select-Object` avant la mise en forme.

### Commandes de reproduction — historique exact non récupéré

Les commandes suivantes permettent de compléter ou reproduire l'inventaire. **Il n'est pas établi que ces lignes exactes ont été utilisées dans les premiers échanges.**

```powershell
# Modèle de la machine
Get-CimInstance Win32_ComputerSystem | Select-Object Manufacturer, Model, TotalPhysicalMemory

# Processeur et contrôleurs graphiques
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors
Get-CimInstance Win32_VideoController | Select-Object Name, AdapterRAM

# Modèles et capacités des disques physiques
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, Size, HealthStatus
Get-Disk | Select-Object Number, FriendlyName, Size, PartitionStyle

# Volumes logiques : capacité et espace libre
Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID, VolumeName, Size, FreeSpace
```

Ne pas interpréter `AdapterRAM` seul comme une mesure fiable de mémoire GPU utilisable par un logiciel d'IA. Le diagnostic matériel n'est pas un test de compatibilité applicative.

## Captures et résultats observés

### RAM et interfaces réseau

![Capture PowerShell : modules mémoire, erreurs corrigées et interfaces réseau](images/ram-et-reseau.jpg)

La capture originale montre :

- Deux modules Kingston, chacun de `4294967296` octets, soit 4 Gio ; référence affichée `ACR24D4S7S1MB-4`.
- Une valeur `Speed` de `1200` pour chaque module. Elle est conservée telle qu'affichée ; aucune mesure indépendante de fréquence n'a été faite.
- Le Wi-Fi Qualcomm Atheros QCA9377 en état `Up`, à `433.3 Mbps`.
- L'Ethernet Realtek PCIe GbE en état `Disconnected`, à `0 bps`. Cela indique une absence de lien lors du relevé, pas une capacité maximale nulle.
- Les essais PowerShell en erreur, puis l'affichage réussi de la mémoire.

La vitesse de liaison Wi-Fi n'est pas un débit mesuré de copie de fichiers. Les deux modules sont affichés avec le même libellé `DIMM 0` : on conserve le résultat sans en déduire une disposition physique précise.

### Captures encore manquantes

Les captures des spécifications Acer, des volumes Windows et des disques physiques ne sont pas accessibles dans l'historique retourné. Elles ne sont ni recréées ni remplacées par des illustrations. Le [registre des images](images/README.md) précise les fichiers à ajouter. Aucun lien d'image actif ne pointe vers un fichier absent.

## Sauvegarde des données

Des fichiers à conserver étaient présents sur C: et D:. Le compte rendu évoquait environ 300 Go au total et environ 180 Go sur le HDD : ce sont des estimations issues des échanges, pas un inventaire de sauvegarde mesuré.

L'utilisateur a déclaré : « C'est bon j'ai transféré mes docs sur un disque dur externe ». Après la consigne d'ouvrir quelques fichiers pour vérifier la copie et de débrancher physiquement ce disque, il a répondu « c'est ok ».

**Ce qui est établi :** le transfert des documents est déclaré réalisé et la consigne de vérification/débranchement a reçu une confirmation globale. Le détail des fichiers copiés, des essais de lecture et de la couverture de toutes les données de C:/D: n'est pas disponible.

Avant le partitionnement, reconfirmer que tous les fichiers souhaités sont sauvegardés, lisibles et que le disque externe est physiquement débranché. Une copie entre C: et D: ne protège pas contre une erreur de sélection touchant les disques internes.

## Prochaine étape : Debian

**Tout ce qui suit est prévu, sans exécution confirmée.**

1. Télécharger une image officielle Debian stable **amd64 netinst**. `amd64` désigne l'architecture PC 64 bits, y compris certains processeurs Intel.
2. Vérifier la somme de contrôle et l'authenticité de l'image selon la documentation Debian.
3. Identifier une clé USB effaçable et préparer le support amorçable depuis Windows, avec Rufus envisagé dans la conversation. Copier simplement le fichier ISO dans la clé ne suffit pas.
4. Démarrer l'Acer sur la clé. Les réglages UEFI et la touche de démarrage seront documentés après observation sur cette machine.
5. Identifier les disques par modèle et capacité. Installer Debian sur le **Kingston 480 Go**, après validation du partitionnement. Le plan prévoit le remplacement de Windows ; son contenu peut être effacé.
6. Décider séparément du sort du **Toshiba 1 To** : conservation ou reconfiguration, après contrôle des données. Aucun formatage automatique n'est présumé.
7. Choisir une installation minimale sans bureau, avec serveur SSH et outils usuels selon les écrans proposés.
8. Brancher Ethernet, relever l'adresse du serveur, puis tester SSH depuis le PC principal.
9. Consigner la version installée, le partitionnement final et le résultat de chaque contrôle avant d'ajouter des services.

La page officielle consultée le 6 octobre 2026 propose Debian **13.7.0 « trixie » amd64 netinst**. Recontrôler la version au téléchargement, plutôt que de figer le nom de l'ISO dans une procédure future. Sources : [téléchargement Debian](https://www.debian.org/download.fr.html) et [préparation du support USB](https://www.debian.org/releases/stable/amd64/ch04s03.fr.html).

Exemple du futur test SSH, **à adapter et non exécuté** :

```powershell
ssh utilisateur@ADRESSE_IP_DU_SERVEUR
```

Le nom de compte et l'adresse réelle seront renseignés après l'installation. `homelab` est un nom envisagé, pas une configuration déjà appliquée.

## Sécurité

> **Écrire une image sur une clé USB efface son contenu. Partitionner ou formater un disque peut détruire ses données. Identifier le support avant de confirmer.**

- Revalider la sauvegarde et débrancher le disque externe avant l'installation.
- Ne pas identifier une cible uniquement par un numéro de disque ou par l'ordre affiché.
- Garder des sauvegardes indépendantes après la mise en service ; NAS et Immich ne seront pas les seules copies des fichiers.
- Prévoir mises à jour, comptes adaptés et contrôle des accès avant d'héberger les services.
- Le plan actuel vise le réseau domestique ; toute exposition à Internet nécessitera une configuration documentée distincte.
- Avant publication, relire les captures : l'original fourni montre des adresses MAC. Masquer les identifiants que l'on ne souhaite pas rendre publics. Ne jamais publier mots de passe, clés privées ou documents personnels.

## Roadmap

- [x] Définir l'objectif de réutilisation du portable.
- [x] Retenir Debian pour une installation minimale.
- [x] Rassembler l'inventaire matériel.
- [x] Relever RAM et interfaces réseau avec PowerShell.
- [x] Copier les documents sur un disque externe, selon confirmation de l'utilisateur.
- [x] Recevoir la confirmation globale après les consignes de vérification et de débranchement.
- [ ] Compléter les captures et les commandes historiques manquantes.
- [ ] Confirmer ISO téléchargée et clé USB disponible.
- [ ] Préparer la clé USB et documenter ses paramètres.
- [ ] Installer Debian sur le SSD et consigner le partitionnement.
- [ ] Contrôler Ethernet et accéder au serveur par SSH.
- [ ] Organiser le HDD et mettre en place un partage NAS/Samba.
- [ ] Définir et tester une stratégie de sauvegarde/restauration.
- [ ] Évaluer puis déployer Immich, selon les ressources et besoins.
- [ ] Évaluer une IA locale et tester son fonctionnement sans Internet.
- [ ] Relire les données publiques et publier le dépôt sur GitHub.

## Utiliser ce projet

```text
homelab/
├── README.md
├── images/
│   ├── README.md
│   └── ram-et-reseau.jpg
└── docs/
    └── modele-etape.md
```

Ouvrir le dossier `homelab` dans VS Code, puis ouvrir `README.md`. Utiliser **Ctrl+Maj+V** pour l'aperçu Markdown. Les images utilisent des chemins relatifs, compatibles avec le dépôt GitHub.

Pour chaque nouvelle opération, copier le [modèle d'étape](docs/modele-etape.md) et renseigner : objectif, commande ou action, explication, résultat réel, capture et validation. Mettre à jour la roadmap uniquement après réalisation.

Ce dossier est prêt à être versionné ; aucun dépôt distant ni aucune publication GitHub n'a été créé par cette documentation.

## Références

- Conversation source : « Serveur maison avec vieux PC » (historique partiellement accessible).
- [Téléchargement officiel de Debian](https://www.debian.org/download.fr.html).
- [Manuel Debian : préparation d'une clé USB](https://www.debian.org/releases/stable/amd64/ch04s03.fr.html).
- [Manuel d'installation Debian amd64](https://www.debian.org/releases/stable/amd64/index.fr.html).

