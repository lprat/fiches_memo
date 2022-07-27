# Etapes importantes lors de la gestion d'un incident grave
Lors de la qualification, si un incident est considéré comme grave (lorsque l'attaquant à pu prendre le contrôle de la majorité du SI), la gestion de l'incident est complexe et va nécessiter l'actions de nombreux acteurs.  

Une compromission majeure indique des nombreuses faiblesses dans la conception et/ou l'exploitation du SI. 
Le CERT Santé a constaté que dans ce type d'incident, il était très souvent plus efficace de porter prioritairement les efforts de remédiation sur la reconstruction du SI afin de corriger ces problèmes de fond et remettre en service un SI "intègre" .

Ainsi, il n’est pas nécessaire d’attendre les résultats des investigations avant de commencer ces travaux mais il est important de réaliser une extraction des preuves à conserver pour l’enquête judiciaire.

## Etapes
<!--ts-->
   * [1 - Confinement](#1---confinement)
   * [2 - Retablir mes services vitaux](#2---r%C3%A9tablir-les-services-vitaux-encore-op%C3%A9rationnels-dans-un-mode-d%C3%A9grad%C3%A9)
   * [3 - Messagerie interne, Communication interne, Communication externe, Communication incident](#3---messagerie-interne-communication-interne-communication-externe-communication-incident)
   * [4 - Reconstruction](#4---reconstruction)
     * [Modèles](#modèles)
     * [Ordre de reconstruction](#ordre-de-reconstruction)
     * [Cycle de reconstruction](#cycle-de-reconstruction)
     * [Respect des procédures dans le temps](#respect-des-procédures-dans-le-temps)
     * [Surface d'attaque](#public)
<!--te-->

## 1 - Confinement

  1. **Déconnecter vos sauvegardes** du réseau puis **vérifier leur état** afin de pouvoir garantir la possibilité de pouvoir récupérer les données de votre SI lors de la reconstruction. Attention, la déconnexion de la sauvegarde n'est pas la déconnexion du serveur de "contrôle" (ex Veeam) mais bien la baie SAN/NAS de stockage où se trouve les données de sauvegarde. Il n'est pas impossible qu'une tache planifiée malveillante existe sur un serveur de contrôle dont le but est de supprimer les sauvegardes à une date ou un événement spécifique (ex: coupure internet).
  1bis. Si un chiffrement du SI est en cours alors **mettre en pause les machines virtuelles** avec des données vitales (DPI, finance, ...), si possible réaliser un "snapshot" (disque et mémoire) afin de stopper le chiffrement en cours. Cela permettra potentiellement de récupérer des informations en mémoire susceptibles d'aider au déchiffrement.
  2. **Couper les accès extérieurs au SI** (internet, connexions avec d'autres sites, partenaires, ...), afin que l'attaquant perde le contrôle "direct" de votre SI. Attention, si vous ne débranchez pas les câbles, vérifiez toutes les sessions initialisées et en cours que le pare-feu laisseraient encore passer (essayer de vous connecter à internet pour confirmer et demander un scan de vos adresses IP publiques à un tiers).
  3. **Protéger vos ressources sur internet** en changeant les secrets (mots de passe, certificats, ...), puis si possible en filtrant les interfaces d'administration à vos adresses IP publiques (vérifier les sessions en cours): DNS externe, site web externalisé, messagerie externalisée [ex: o365 mise en place d'accès conditionnel], accès finance/banque, serveurs externes, ... Afin que l'attaquant ne puisse pas prendre le contrôle de vos ressources externes par des secrets qu'il aurait volés sur votre SI.

## 2 - Rétablir les services vitaux encore opérationnels dans un mode dégradé

Prérequis:  Le pare-feu d'interconnexion avec internet ne doit pas avoir été compromis et sa sécurité doit être forte (mise à jour de sécurité, pas d'exposition d'interface sur internet, ...).  

  1. Faire **l'inventaire des services vitaux** du SI en vérifiant leur état opérationnel (HS ou Opérationnel [potentiellement compromis mais non impacté] ).
  2. Sur les services opérationnels, faire un snapshot (si cela n'a pas été réalisé dans la phase confinement) puis identifier si les services du point 1.1 ont besoin d'ouverture de flux vers internet/partenaires en notant les adresses IP, ports et protocoles (communiquer avec l'éditeur pour connaître la matrice de flux).
  3. **Ouverture des flux strictement nécessaires** au fonctionnement des services vitaux en **point à point** exclusivement sur les protocoles qui ne sont pas à risque (aucun flux de contrôle/administration: RDP, SSH, RAT, SMB, RPC, ...) avec journalisation.
 
## 3 - Messagerie interne, Communication interne, Communication externe, Communication incident

1. Si la messagerie est interne (sur site) :  
  * et que votre serveur MX est interne mais celui-ci n'a pas été compromis par l'incident alors laisser l'ouverture du flux exclusivement en entrée (attention interdiction pour le serveur de sortir vers internet, ainsi que distribuer vers le serveur de messagerie interne), adapter la rétention du MX, afin de limiter la perte de courriels ; Dans le cas où la messagerie n'a pas été compromise, alors il est possible selon le contexte de distribuer les courriels reçus à condition d'enlever toutes les pièces jointes.  
  * et que votre serveur MX est interne mais celui-ci a été compromis par l'incident, alors voir pour externaliser le service jusqu'à résolution de l'incident (redirection MX sur votre DNS externe) ;  
  * et que votre serveur MX est externe, alors contacter votre prestataire MX pour lui demander de mettre en place une rétention MX importante jusqu'à la réouverture de votre SI, afin de limiter la perte de courriels.  
2. Il est important de pouvoir communiquer en interne pour faciliter la gestion de l'incident. Il existe plusieurs solutions pour permettre cette communication, soit :  
  * votre messagerie est externe, vous avez réinitialisé tous les comptes lors du confinement (potentiellement vous avez activé une authentification multi-facteurs) lors de l'étape de confinement, vous pouvez utiliser ce canal pour vos communications avec des smartphones (réseau GSM) ;
  * votre messagerie est interne et celle-ci est toujours opérationnelle malgré l'incident, vous pouvez utiliser ce canal pour vos communications internes ;  
  * votre messagerie est interne mais elle a été impactée et n'est plus opérationnelle. Vous pouvez utiliser avec des smartphones des plateformes en ligne comme "Tchap", "Slack", ... Si vous disposez d'un serveur linux opérationnel et non compromis, vous pouvez installer une solution interne à déploiement rapide: MailCow (https://mailcow.email/), MailU (https://github.com/Mailu/Mailu) ...  
3. Il est parfois important de pouvoir communiquer vers l’extérieur. Vous avez plusieurs solutions en fonction de votre architecture :  
  * votre messagerie est externe, vous avez réinitialisé tous les mots de passe lors du confinement (de préférence, vous avez aussi activé une authentification multi-facteurs lors de l'étape de confinement), vous pouvez utiliser ce canal pour vos communications avec des smartphones (réseau GSM) ;  
  * votre serveur de messagerie était interne, il est préférable de créer des boites temporaires à l'aide de prestations externes (prestataire spécialisé ou un service grand public: Protonmail, Gmail, ... selon vos contraintes).  
4. Il est important d'identifier un plan de communication par rapport à votre incident (pour l'interne, pour l’extérieur, proactif ou réactif).  
 
 ## 4 - Reconstruction
 ### Modèles
 |Nom|Description|Avantages|Inconvénients|
 |------|------|------|-----|
 |"zéro trust"|consiste à réinstaller dans une zone sécurisée d'installation chaque équipement/machine (selon un ordre établi de dépendance), les durcir afin de sécuriser toute sa surface d'exploitation possible, puis le remettre dans le SI compromis au fur et à mesure avec la certitude qu'il ne pourra plus être compromis.| On réinstalle au fur et à mesure le SI sans avoir besoin de nombreux serveurs en réserve (potentiellement nécessaire pour la partie hyperviseur et sauvegarde). On simplifie l'organisation car il n'y a pas de problématique liée à la possession de services dans des zones différentes demandant d'avoir plusieurs postes par personne (sur chaque SI propre ou infecté).|Nécessite une expertise en sécurité importante et du temps pour que chaque flux soit maitrisé (micro segmentation).|
 |"Bulle propre"|consiste à créer un nouveau SI séparé de l'ancien compromis (sans aucune possibilité de discussion entre les deux).|Nécessite moins d'expertise technique.|Cette technique oblige à avoir des serveurs/équipements/postes en réserve (proche du double) afin de continuer à faire fonctionner l'ancien tout en créant le nouveau SI. Cela veut dire aussi, que certains services seront sur le nouveau SI pendant que d'autres seront encore sur l'ancien, cela nécessite une organisation importante.|

### Ordre de reconstruction
Il est important de reconstruire un SI selon un ordre bien défini (dépendance). La cartographie du SI devrait pouvoir permettre d'identifier les dépendances.  
Veuillez trouver un exemple de dépendances classiques:
 1. Postes administrateurs dédiés à l'administration (avec Keepass/certificat) qui permettront la réinstallation du nouveau SI (poste durci, sans bureautique, sans dependance de controle par un niveau inferieur => hors AD ?)
 2. Firewall et routeur pour créer la zone d'installation/durcissement
 3. Hyperviseurs, serveurs physiques, baies de stockage
 4. Services socles informatiques vitaux : sauvegarde, supervision, DNS, ...
 5. Services socles informatiques sécurité : bastion, serveurs d'authentification et d'autorisation, antivirus, serveurs de mise à jour, serveur IGC/PKI, journalisation centrale, proxy sortant, reverse proxy, ...
 6. Services métiers vitaux (selon la structure, ex: DPI/patients, finance, ...)
 7. Services socles informatiques importants: DHCP, banc installation postes, serveurs de configuration, CMDB, DFS, ...
 8. Services métiers importants (selon la structure)
 9. Postes de travail

### Cycle de reconstruction

1. Choix sur l'origine de l'équipement/machine/service à reconstruire selon l'ordre de dépendance (une option uniquement) :
  * Installation/Réinstallation de zero (from scratch)
  * Récupération d'une machine ou un équipement compromis (vérifier par un expert en sécurité) depuis une sauvegarde, VM, équipement ... Sur le SI infecté (compromis)
  * Récupération sur une machine ou un équipement compromis, des données encore viables qui ne sont pas dangereuses (pas de PE/fichier type webshell/configuration susceptible de contenir une modification malveillante ... À vérifier par un expert en sécurité) depuis une sauvegarde, VM, équipement ... Sur le SI infecté (compromis)
  * Récupération depuis une sauvegarde saine (interne, externe, point de restauration), nécessite d'avoir identifié avec l'investigation la date exacte du début de la compromission afin de choisir une sauvegarde saine.
1.bis - La récupération d’éléments depuis le réseau infecté doivent être manipulés sur un réseau complètement isolé par un expert en sécurité qui vous transmettra les données sur un support "propre" autorisé à entrer en zone d'installation  
2. Installation dans une zone dédiée à l'installation (sans lien avec le SI compromis)  
3. HARDWARE
  * Installation / Récupération hardware (selon le choix 1) : si récupération alors réinitialiser la configuration hardware (ex: BIOS, disques, ...)
  * Durcissement hardware: Appliquer les mises à jour, activer les options de sécurité du BIOS (selon la sécurité physique du lieu), activer les options RAID adaptées, reconfigurer l'outil interne BIOS / firmware de gestion des serveurs à distance (ex: ILO HP) en le désactivant ou sécurisant son accès ...
    * Attention au choix du referentiel d'authentification et d'autorisation afin de ne pas utiliser un referentiel permettant le control sur le hardware avec un niveau de criticité inferieur à votre service, privileger une (authentification forte)[connexion_auth_sec.md]
  * test
4. Système d'exploitation (OS)  
  * Installation / Récupération du système d'exploitation ou équipement: privilégier l'installation minimaliste (Windows core, Linux minimal)  
  * Durcissement du système d'exploitation / équipement : Respecter les guides de l'ANSSI, utiliser le support du CERT Santé (https://cybersante.github.io/CERT_Sante_Accompagnement/ en atteignant le niveau 3 minimum dans le scénario "bulle SI propre" et niveau 4 minimum pour "zéro trust").  
    * Attention au choix du referentiel d'authentification et d'autorisation afin de ne pas utiliser un referentiel permettant le control sur le système avec un niveau de criticité inferieur à votre service, privileger une (authentification forte)[connexion_auth_sec.md]  
  * Micro-segmentation des flux de dépendance "OS" : administration (ssh, rdp, wmi, psh, psexec), supervision, AV, ...  
5. Service/Rôle  
  * Installation / Récupération du service / rôle  
  * Durcissement du service / rôle : Respecter les guides de l'ANSSI, utiliser le support du CERT Santé (https://cybersante.github.io/CERT_Sante_Accompagnement/ en atteignant le niveau 3 minimum dans le scénario "bulle SI propre" et niveau 4 minimum pour "zéro trust"), utiliser des outils d'audit adaptés comme indiqué dans le support (ex: Pingcaslte/ORADAD pour l'AD, Nmap pour vérifier la surface d'exploitation ...)  
    * Attention au choix du referentiel d'authentification et d'autorisation afin de ne pas utiliser un referentiel permettant le control sur le service avec un niveau de criticité inferieur à votre service, privileger une (authentification forte)[connexion_auth_sec.md]
  * Micro-segmentation des flux pour le service / rôle : si le service ne peut être appelé que par une quantité limitée d'adresses IP internes, alors filtrer en local (ex: service web dont les requêtes ne peuvent provenir que du reverse proxy)  
6. Filtrage interzone  
  * Zones internes : Ouverture des flux interzones internes (DMZ in, DMZ out, serveurs, admin, postes, ....) nécessaires au fonctionnement du service / rôle  
  * Zones d'interconnexion internet ou partenaires : Ouverture des flux d'interconnexion sécurisés entrants / sortants vers partenaire, internet ... Nécessaire au fonctionnement du service / rôle  
7. Déplacement du service installé sur le SI bulle ou zero trust  
 
### Respect des procédures dans le temps
 
Après un incident, les procédures et mecanismes de sécurité seront très fortement renforcés.  
Il est nécessaire que les informaticiens (administrateurs, support, ...) continuent à maintenir le niveau de sécurité du SI en appliquant ces procédures malgré les contraintes qu'elles peuvent procurer par rapport à l'ancien fonctionnement.  
Sans cela, le SI redeviendra rapidement vulnérable et les efforts auront été vains.

### Surface d'attaque
![image](https://user-images.githubusercontent.com/1132448/181287786-619b78d7-5093-4ab9-8fed-b85c1a9c1562.png)
