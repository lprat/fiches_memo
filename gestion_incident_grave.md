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
     * [Réseau et interconnexion](#r%C3%A9seau--interconnexion)
     * [Cycle de reconstruction](#cycle-de-reconstruction)
     * [Carthographie](#carthographie)
     * [Procédures organisationnelles](#procédures-organisationnelles)
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
 |"zéro trust"|consiste à réinstaller dans une zone sécurisée d'installation chaque équipement/machine (selon un ordre établi de dépendance), les durcir afin de sécuriser toute sa surface d'exploitation possible, puis le remettre dans le SI compromis mais sur des VLAN avec ACL (où seul des machines assainies/propres seront présentes) au fur et à mesure avec la certitude qu'il ne pourra plus être compromis.| On réinstalle au fur et à mesure le SI sans avoir besoin de nombreux serveurs en réserve (potentiellement nécessaire pour la partie hyperviseur et sauvegarde). On simplifie l'organisation car il n'y a pas de problématique liée à la possession de services dans des zones différentes demandant d'avoir plusieurs postes par personne (sur chaque SI propre ou infecté).|Nécessite une expertise en sécurité importante et du temps pour que chaque flux soit maitrisé (micro segmentation).|
 |"Bulle propre"|consiste à créer un nouveau SI séparé de l'ancien compromis (sans aucune possibilité de discussion entre les deux).|Nécessite moins d'expertise technique.|Cette technique oblige à avoir des serveurs/équipements/postes en réserve (proche du double) afin de continuer à faire fonctionner l'ancien tout en créant le nouveau SI. Cela veut dire aussi, que certains services seront sur le nouveau SI pendant que d'autres seront encore sur l'ancien, cela nécessite une organisation importante.|
 
### Ordre de reconstruction
Il est important de reconstruire un SI selon un ordre bien défini (dépendance). La cartographie du SI devrait pouvoir permettre d'identifier les dépendances.  
Veuillez trouver un exemple de dépendances classiques:
 1. Postes administrateurs dédiés à l'administration (avec Keepass/certificat) qui permettront la réinstallation du nouveau SI (poste durci, sans bureautique, sans dependance de controle par un niveau inferieur => hors AD ?). Possibilité d'installer une machine virtuelle pour avoir un poste bureautique.
 2. Firewall et routeur pour créer les zones "installation/durcissement" ainsi que les nouveaux VLAN avec ACL pour les machines assainies
 4. Hyperviseurs, serveurs physiques, baies de stockage
 5. Services socles informatiques vitaux : sauvegarde, supervision, DNS, ...
 6. Services socles informatiques sécurité : bastion, serveurs d'authentification et d'autorisation (IAM, AD, ...), antivirus, serveurs de mise à jour, serveur IGC/PKI, journalisation centrale, proxy sortant, reverse proxy, ...
 7. Services métiers vitaux (selon la structure, ex: DPI/patients, finance, ...)
 8. Services socles informatiques importants: DHCP, banc installation postes, serveurs de configuration, CMDB, DFS, ...
 9. Services métiers importants (selon la structure)
 10. Postes de travail

### Réseau / Interconnexion
Références: 
  - https://www.ssi.gouv.fr/uploads/2012/01/anssi-guide-passerelle_internet_securisee-v2.pdf
  - https://www.ssi.gouv.fr/uploads/2022/02/anssi-guide-recommandations_configuration_commutateurs_pare-feux_hirschmann.pdf#chapter.4
#### Interconnexion Internet
L'accès à internet représente un risque important pour un SI. Il est important de le maitriser en controlant les flux par un "sas d'entrée" (dmz in) ainsi que par un "sas de sortie" (dmz out). Cette solution permet de simplifier les matrices de flux qui resteront toujours des règles de filtrage en "entrée" sur la zone, on respectera toujours un filtrage au plus fin. De plus, on limitera au maximum les flux entre les dmz et l'interne, on privilegira toujours l'utilisation de technologie "reverse proxy" pour aller de la "dmz in" vers internet, et de proxy dans la "dmz out" pour aller de l'interne vers internet, afin de pouvoir inspecter et controler les flux entrant et sortant.

#### Interconnexion type IPSEC ou Réseau privé distant
On considerera ce type d'interconnexion aussi dangereuse qu'une interconnexion avec internet car le ou les partenaires présents sur ce réseau ont souvent leur propre accès internet, et un attaquant pourrait accèder à votre SI à partir de ce tiers.

Il est donc obligatoire d'avoir une matrice de flux en entrée la plus fine possible (si possible flux point à point avec port spécifique). Interdire l'accès à l'annuaire AD, et si possible privilégié l'utilisation de reverse proxy comme pour la zone "dmz in".

#### VLAN / Segmentation

Les règles sur la partie réseau à respecter:
  * Il faut utiliser du VLAN de port physique pour une sécurité importante (le filtre par l'adresse MAC est très facilement falsifiable).
  * Il est obligatoire de filtrer par des ACL en entrée tous les VLAN. 
  * Dans le cas des VLANs critiques (DMZ in, DMZ out, app obsolètes, serveurs, ...), il est important d'utiliser une patte firewall disponible.
  * Limiter l'utilisation de port en mode "trunk" car si le port vient à être controlé par un attaquant alors il pourra accèder à tous vos VLANs.
  * Sur un poste d'admin avec une machine virtuelle bureautique, il est préférable d'avoir deux cartes réseaux, une pour l'hote admin sur le VLAN admin et une pour la machine virtuelle bureautique sur le VLAN bureautique.  
  * S'il n'est pas possible d'activer un firewall local (micro segementation) sur une machine critique/importante, alors il sera préférable de la mettre dans une zone (vlan) seule pour limiter le risque de latéralisation à cause d'un serveur compromis dans la même zone (vlan).
  * Penser à désactiver les ports non utilisé
  * Activer l'option "port security" sur les ports (reduira le risque d'usurpation par adresse MAC ainsi que des postes qui virtualiserait des machines sans autorisation).
  * Activer le DHCP Snooping (ainsi que l'option 82 du le DHCP) pour limiter le risque d'usurpation de DHCP et identifier par le DHCP le port de connexion des postes de travail
  * Activer l'ARP Inspection pour limiter l'usurpation d'adresse IP
  * Si une prise réseau est dans un endroit à risque (passage de public, ...) alors il peut etre important d'activer l'authentification par i802.1x 
  
Ci-dessous un exemple de VLAN possibles:  
 |Nom|rôle|
 |------|------|
 |Postes d'administration|Rendre impossible l'usurpation d'un poste d'admin dans le SI et pouvoir filtrer l'accès au bastion ou services d'administration à ce VLAN|
 |bastion ou rebond d'administration|Rendre impossible l'usurpation d'un bastion ou rebond d'admin dans le SI et pouvoir filtrer l'accès aux services d'administration à ce VLAN|
 |Administration des équipements réseau(fw,router,switch)|Dédié un port de l'équipement à son administration qui sera dans ce VLAN. Autoriser l'accès à ce VLAN seulement au bastion ou VLAN "postes d'administration"|
 |Administration des serveurs par carte de gestion à distance (ilo/idrack/...)|la carte doit être mis dans un VLAN dédié. Autoriser l'accès à ce VLAN seulement au bastion ou VLAN "postes d'administration"|
 |Serveurs DMZ in (MX, webmail, vpn, webmail, NS, ...)|Tous les flux en provenance d'internet arrivent dans cette zone, cela permettra de limiter et de controler de manière fine les flux qui pourront aller vers l'interne. |
 |Serveurs DMZ out (smtp out, proxy, dns out, ...)|Tous les flux sorant vers internet doivent passer par cette zone.|
 |Sauvegarde|Le serveur est sur un VLAN dédié car il est critique. Autoriser l'accès à ce VLAN seulement au bastion ou VLAN "postes d'administration"|
 |bais de stockage (SAN/NAS)|La carte d'administration doit être mis dans un VLAN dédié. Autoriser l'accès à ce VLAN seulement au bastion ou VLAN "postes d'administration"|
 |Hyperviseur(s)|Souvent il aura un port trunk afin de pouvoir mettre les VM sur le bon VLAN. La partie Le serveur est sur un VLAN dédié car il est critique. Autoriser l'accès à ce VLAN seulement au bastion ou VLAN "postes d'administration"|
 |Serveurs socle (syslog, supervision, auth, inventaire, DNS relai, smtp relai, ...)|Controleur l'accès aux services socles ainsi que leurs accès dans les autres VLANs|
 |Serveurs BDD|Permet de limiter l'accès sur le port de la base aux services metiers et filtrer l'accès admin aux VLAN d'administration|
 |Serveurs Metiers|Controleur l'accès aux services/applications metiers|
 |Imprimantes|Permet de les identifier pour limiter leurs accès dans le SI et vers internet|
 |IOT(sonde, badge, incendie, ...)|Permet de limiter les interactions avec les autres VLAN.|
 |Serveurs Bureautique(AD, DFS, DHCP, DNS, ...)||
 |Serveurs à risque (boite noire, editeurs à risque, ...)|Controler l'accès des serveurs à risque|
 |Postes de travail|Permet de les identifier pour limiter leurs accès dans le SI|
 |Prestataires|Limiter l'accès pour les prestataires au bastion prestataire ou serveur dédié|

**Utiliser de préférence un langage abstrait pour la creation de votre matrice comme "caprica".**

### Cycle de reconstruction

Télécharger [un exemple de fiche check list](https://form.jotform.com/222611416670348).  


1. **Choix sur l'origine de l'équipement/machine/service à reconstruire selon l'ordre de dépendance** (une option uniquement) :
  * Installation/Réinstallation de zero (from scratch)
  * Récupération d'une machine ou un équipement compromis (vérifier par un expert en sécurité) depuis une sauvegarde, VM, équipement ... Sur le SI infecté (compromis)
  * Récupération sur une machine ou un équipement compromis, des données encore viables qui ne sont pas dangereuses (pas de PE/fichier type webshell/configuration susceptible de contenir une modification malveillante ... À vérifier par un expert en sécurité) depuis une sauvegarde, VM, équipement ... Sur le SI infecté (compromis)
  * Récupération depuis une sauvegarde saine (interne, externe, point de restauration), nécessite d'avoir identifié avec l'investigation la date exacte du début de la compromission afin de choisir une sauvegarde saine.
1.bis - **La récupération d’éléments depuis le réseau infecté** doivent être manipulés sur un réseau complètement isolé par un expert en sécurité qui vous transmettra les données sur un support "propre" autorisé à entrer en zone d'installation  
2. **Installation dans une zone dédiée à l'installation** (sans lien avec le SI compromis)
  * Rappel important: tous les secrets doivent changer à chaque installation, sur les secrets de type "mot de passe" une politique forte doit être appliquée, sur la partie certificat il faut révoquer les anciens certificat, ...   
4. **HARDWARE**
  * Installation / Récupération hardware (selon le choix 1) : si récupération alors réinitialiser la configuration hardware (ex: BIOS, UEFI, disques, ...)
  * Durcissement hardware:
    * Appliquer les mises à jour des firemwares (toujours télécharger sur le site officiel - vérifier les hashs)
    * Activer les options de sécurité du BIOS/UEFI (selon la sécurité physique du lieu)
    * Activer les options RAID adaptées (pour serveur)
    * Dans le cadre d'un serveur, s'il possède une carte de gestion à distance (ex: ILO HP/Idrack dell/...), sécurer son accès
  * Faire controler l'application des mesures ci-dessus par un tiers ayant les qualifications nécessaires en sécurité
5. **Système d'exploitation** (OS)  
  * Installation / Récupération du système d'exploitation ou équipement: privilégier l'installation minimaliste (Windows core, Linux minimal) 
    * Il est possible de mettre en place des mécanismes pour automatiser la création de VM sécurisée selon votre politique, car globalement, il s'agit d'un socle qui change peut entre deux serveurs (seulement la RAM/CPU/Disques). Ex: terraform/Cobbler + ansible
  * Dans le cadre d'un serveur dans un data center externalisé ou dans une salle serveur mal sécurisée:
    * Chiffrer les disques 
    * Interdire le plug d'équipement à chaud (USB)
    * Fixer les drivers autorisés et interdire le chargement d'autres drivers
    * Interdire le montage automatique (CD/DVD/USB/...)
  * Dans le cadre d'un poste de travail ou admin "portable":
    * Si l'utilisateur à un disque dur externe, le chiffrer aussi (même dans le cadre d'un poste fixe)
  * Durcissement du système d'exploitation / équipement selon [la check list](https://form.jotform.com/222611416670348) basée sur https://cybersante.github.io/CERT_Sante_Accompagnement/
  * Micro-segmentation des flux de dépendance "OS" : administration (ssh, rdp, wmi, psh, psexec), supervision, AV, ...  Mettre à jour la matrice de flux.
  * Faire controler l'application des mesures ci-dessus par un tiers ayant les qualifications nécessaires en sécurité
6. **Service/Rôle** (serveur)
  * Installation / Récupération du service / rôle  
    * Si possible privilégier l'utilisation d'installation automatique du service (hors données persistantes) par ansible / conteneur en maitrisant la partie secret pour qu'elle ne soit pas présente dans un dépot.
  * Durcissement du service / rôle selon [la check list](https://form.jotform.com/222611416670348) basée sur https://cybersante.github.io/CERT_Sante_Accompagnement/
  * Faire controler l'application des mesures ci-dessus par un tiers ayant les qualifications nécessaires en sécurité
7. **Filtrage interzone**
  * Zones internes : Ouverture des flux interzones internes (DMZ in, DMZ out, serveurs, admin, postes, ....) nécessaires au fonctionnement du service / rôle  
  * Zones d'interconnexion internet ou partenaires : Ouverture des flux d'interconnexion sécurisés entrants / sortants vers partenaire, internet ... Nécessaire au fonctionnement du service / rôle  
  * Faire controler l'application des mesures ci-dessus par un tiers ayant les qualifications nécessaires en sécurité
8. **Déplacement du service installé sur le SI bulle ou zero trust** 
 

#### Spécificité du serveur de sauvegarde
La sauvegarde est une partie critique du SI, elle contient toutes les données du SI dont ses secrets, il est donc important de respecter des règles spécifiques:
  - Utiliser de préférence un référentiel d'authentification et d'autorisation spécifique
  - Chiffrer la sauvegarde (obligatoire si elle est déportée en dehors du SI, ex: cloud)
  - Si possible avoir une sauvegarde hors réseau (ex: sur bande)
  - Si les sauvegardes sont poussées sur le serveur alors deplacer les dès reception en changeant les droits d'accès pour les rendrent non accessibles à l'agent qui les pousse. 

#### Spécificité de l'hyperviseur
Le durcissement de l'hyperviseur doit être vu comme un service. Il est important que seul les administrateurs du SI est accès à la console d'administration.  
Dans le cadre où un utilisateur spécifique à le droit d'administrer un serveur sur l'hyperviseur, il est important de ne lui donner que des droits sur ce serveur spécifique. De plus, cette utilisateur devra respecter l'utilisation d'une authentification forte.

#### Spécificité du controleur de domaine
L'Active directory est souvent la base de l'authentification et de la gestion de configuration du parc bureautique. Il s'agit d'un brique très importante pour garantir la sécurité du réseau de bureautique mais aussi très complexe.  
Lors de la compromission d'un domaine AD, il est important de realiser une operation de "bascule" qui à pour but d'enlever toutes persistances de l'attaquant que cela soit sur le système d'exploitation ou dans la partie AD.   
Cette operation doit être realisée par des personnes formées à cette methode et certifiées (PASSI / PRIS).  

#### Spécificité du poste de travail
Ces préconisations ne sont pas pour les postes d'admin qui n'ont pas accès à internet (passage par la machine virtuelle).  
Actions de durcissement spécifiques (ref.: https://cybersante.github.io/CERT_Sante_Accompagnement/):
  * Configuration des lecteurs multimedia avec les options de durcissement en fonction de l'editeur
  * Déinstallation des lecteurs multimedia non nécessaires
  * Mettre en place une politique d'execution (interdire l'execution en dehors des chemins windows => c:\windows\ et c:\program files\)

#### Spécificité maintenance partage d'ecran depuis poste admin
Il est parfois nécessaire de pouvoir partager son ecran dans le cadre de l'assistance à la maintenance. Certains bastion ont cette option par défaut. Afin de realiser cette action de manière sécurisée, il est interessant de mettre en place une solution de type partage d'ecran par le web en DMZ accessible seulement à vous en interne et votre prestataire.  

Par exemple , il existe une solution opensource basée sur webrtc: https://github.com/muaz-khan/WebRTC-Experiment#webrtc-screen-sharing

### Carthographie
La cartographie du SI doit permettre de connaitre la criticité de chaque service ainsi que les matrices suivantes:
  - Matrice d'accès d'administration: utilisateur -> auth & autorisation (AD / IAM / BASTION) -> service|système d'exploitation|équipement réseau
  - Matrice d'accès service: utilisateur -> auth & autorisation (AD / IAM / BASTION) -> service
  - Matrice de flux: service|système d'exploitation -> accès internet|accès interne
  - Matrice de dépendance service|système d'exploitation:
        - dependance d'authentification pour le controle
        - dependance hardware: container/virtuelle/physique
        - Dependance de lancement du service: root/user
        - dépendance socle: agent/service socle de controle

Il existe des outils open-source interessants pour realiser la cartographie comme "netbox".  

### Procédures organisationnelles

Les procédures organisationnelles sont là pour permettre de maintenir le SI dans un état sécurisé dans le temps.  

Liste non exaustive des procédures importantes:
  - Procédure de surveillance des CVE critiques (https://www.cisa.gov/known-exploited-vulnerabilities-catalog & https://www.cert.ssi.gouv.fr/alerte/) et d'application sur le SI
  - Procédure d'administration des serveurs (bonnes pratiques pour éviter le vol d'identifiants, utilisation de clé FIDO2, ...)
  - Procédure pour maintenir la cartographie à jour (automatisation possible)
  - Procédure d'installation d'un nouveau service
  - Procédure contenant une check list de points techniques à verifier sur le SI pour assurer que la politique de sécurité est respectée (scan du SI, passage de pingcastle, utilisation de spyre, analyse des logs, ...)
  - Procédure de gestion de l'obsolescence
  - Procédure de départ et d'arrivée d'une personne de la DSI
  - Procédure de départ et d'arrivée d'un utilisateur simple
  - ...

### Respect des procédures dans le temps
 
Après un incident, les procédures et mecanismes de sécurité seront très fortement renforcés.  
Il est nécessaire que les informaticiens (administrateurs, support, ...) continuent à maintenir le niveau de sécurité du SI en appliquant ces procédures malgré les contraintes qu'elles peuvent procurer par rapport à l'ancien fonctionnement.  
Sans cela, le SI redeviendra rapidement vulnérable et les efforts auront été vains.

### Surface d'attaque
![image](https://user-images.githubusercontent.com/1132448/181287786-619b78d7-5093-4ab9-8fed-b85c1a9c1562.png)
