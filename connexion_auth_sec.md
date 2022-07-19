# Connexion authentifiée et sécurisée entre un client et un service

Une connexion authentifiée et sécurisée entre un client et un service va l'utiliser les différentes possibilités offertes par les algorithmes cryptographiques.  
La majorité des protocoles de sécurité moderne (TLS, SSH, ...) possèdent plusieurs étapes :
 1. Génération de secrets (le prérequis, sans secret pas d'authentification) ;
 2. Transmission au client et au serveur de leur secret propre ainsi qu'un élément en lien avec le secret de la partie adverse lui permettant de confirmer son identité (ex: Certificat public ou Certificat CA pour vérifier la signature du certificat transmis) ;
 3. l'authentification qui va permettre de vérifier l'identité du service (pour le client) et du client (pour le service) ;
 4. le partage d'un secret après l'authentification permettant de créer une communication qui ne pourra jamais être déchiffrée (même si un jour le secret d'authentification est compromis) car la clé est unique et éphémère ;
 5. Appliquer les autorisations sur le service en fonction de l'utilisateur authentifié ;
 6. Création de la communication chiffrée à partir du secret commun ;
 7. Le rafraichissement/réinitialisation du secret commun ;
 8. La déconnexion (selon une durée de validité du secret commun, une action de déconnexion du client, ou une inactivité trop longue)

Les étapes 3 à 5 sont souvent nommées "poigné de main" (handshake).

On peut distinguer 2 grands mécanismes de cryptographie :
  - Symétrique qui permet :
    - chiffrement par bloc (ex: AES ou tripe-DES) ou flot (Chacha20)
    - fonctions de hachage (ex: SHA-2 ou SHA-3)
  - Asymétrique (RSA, FF-DLOG, EC-DLOG) qui permet :
    - chiffrement asymétrique (ex: RSA-OAEP, ECIES-KEM et DLIES-KEM)
    - signature électronique (ex: PKCS, EC-DSA, EC-KDSA)
    - échange de clé (ex: Diffie-Hellman [DH] ou Courbe Eliptique Diffie-Hellman [EC-DH])

Reference : https://www.ssi.gouv.fr/guide/mecanismes-cryptographiques/

Lors de l'utilisation du mécanisme asymétrique pour l'authentification, 2 possibilités sont offertes pour prouver l'identité :
  - le client transmet un secret aléatoire chiffré avec la clé publique et le service l'utilise pour initialiser la connexion (s'il n'a pas la clé privée alors il ne peut pas déchiffrer le secret et donc initialiser la connexion)
  - le service signe une valeur aléatoire transmise par le client permettant de prouver qu'il possède la clé privée

L'authentification avec un mécanisme symétrique nécessite forcément un secret commun (comme on peut le voir avec SSH ou TLS lors de la fin de la poigné de main où le protocole utilise un chiffrement). Exemple le protocole PAKE ou Kerberos nécessite le partage d'un secret entre les deux parties en amont. Dans le cas du Kerberos, [le secret partagé est un hash](https://redmondmag.com/articles/2012/02/01/understanding-the-essentials-of-the-kerberos-protocol.aspx) qui selon la configuration peut être le [NT Hash](https://blog.gentilkiwi.com/securite/mimikatz/overpass-the-hash).  

Si on reprend l'exemple du SSH
```
 - Choix de type de secrets entre "mot de passe" et certificat ("ssh -Q key"):
ssh-ed25519
ssh-rsa
ssh-dss
...
 - Choix du partage de secret commun ("ssh -Q kex"):
diffie-hellman-group1-sha1
ecdh-sha2-nistp256
curve25519-sha256
...
 - Choix du chiffrement de la connexion finale ("ssh -Q cipher")
3des-cbc
aes256-cbc
aes256-ctr
chacha20-poly1305@openssh.com
...
 - Choix du mécanisme de détection d'une modification des messages ("ssh -Q mac")
hmac-sha1
hmac-sha2-256
hmac-sha2-512
...

Le serveur SSH utilise une clé publique/privée nommée "host key" (clé publique/privée en RSA, ECDSA et AD25519 - stocké dans le repertoire "/etc/ssh/"), qui lui permet de garantir son identité par rapport au client.

L'ordre de la poigné de main en SSH est la suivante :
 - Connexion depuis un client vers le service SSH en indiquant sa version SSH
 - Le serveur répond en indiquant sa version SSH
 - Le client indique les algorithmes supportés pour l'échange de clé, de chiffrement et de vérification d'une modification de message
 - Le serveur indique les algorithmes supportés pour l'échange de clé, de chiffrement et de vérification d'une modification de message
 - Le client échange sur la valeur diffie hellman choisie
 - Le serveur transmet la valeur diffie hellman choisie et signe les deux (la sienne et celle du client) avec sa clé privée
 - Le client vérifie la validité de la signature et lui permet d'authentifier le serveur
 - La communication chiffré selon le protocole négocié basé sur le secret commun généré par diffie-hellman
 - Pour vérifier l'identité du client :
   - S'il s'agit d'une authentification par certificat alors le serveur lui transmet un message aléatoire qu'il doit signer avec sa clé privée pour prouver de son identité
   - s'il s'agit d'un mot de passe alors il est chiffré et utilise le mécanisme "MAC" négocié pour le transmettre afin qu'il soit vérifié selon le protocole défini pour vérifie un mot de passe local (globalement : hachage du mot de passe et vérification que le hash local est identique à celui calculé avec le secret transmis).
 - Si l'authentification réussie alors le dialogue peut continuer et le service SSH fourni les droits d'accès liés à l'utilisateur authentifié.
 
Reference : https://slidetodoc.com/transportlevel-and-web-security-ssl-tls-ssh-chapter/
```

### Schéma d'une authentification authentifiée et sécurisée

![image](https://user-images.githubusercontent.com/1132448/178784789-820351fa-67aa-4d52-854d-7c7093867f66.png)

## La protection des secrets

Les secrets sont indispensables à l'authentification.  
Il est donc important de prendre en compte certains points :
  - Comment on crée le secret pour assurer qu'il est conforme au niveau de sécurité d'authentification souhaité ;
  - Où sera situé le secret et comment assurer sa confidentialité et son intégrité (client, service, générateur du secret, ...) ;
  - Comment on transmet le secret pour assurer sa confidentialité et son intégrité du secret ;
  - Cycle de vie d'un secret (durée de vie, peut-on allonger la durée de vie, comment on peut réinitialiser ou révoquer le secret, ...) ;

La transmission des secrets aux différentes parties et l'enrôlement dans le cadre de l'authentification symétrique augmente la surface d'attaque.
Dans le cadre de l'authentification asymétrique, l'enrôlement n'induit pas de risque de perte de secret, la partie transmissions du secret reste là encore un point important où il faut être vigilant.
Un secret basé sur un algorithme asymétrique à moins de surface d'attaque qu'un secret basé sur une algorithme symétrique.

En plus de sa caractéristique d'algorithme cryptographique, on peut ranger un secret selon son **facteur** d'authentification :
 - facteur de connaissance : Connaissance mémorisable => ex: mot de passe
 - facteur de possession : élément secret non mémorisable qui peut être contenu :
   -  dans un composant de sécurité (doit protéger cet élément de toute extraction) => ex: carte à puce, clé [FIDO2](https://fidoalliance.org/certification/authenticator-certification-levels/), ...
   -  en dehors d'un composant de sécurité => ex: certificat privé et public sur un poste de travail
 - facteur inhérent : caractéristique physique intrinsèquement liée à une personne et indissociable de la personne elle-même => ex: empreinte, rétine, ...

Quand on parle d'une authentification multi facteurs, il s'agit d'employer des facteurs d'authentification (ci-dessus) différents, et non pas l'utilisation d'une catégorie de facteur unique plusieurs fois (ex: 2 mots de passe ou 2 TOKEN).

La vérification du multi facteurs peut être située à plusieurs niveaux :
 - L'hôte distant vérifie les deux facteurs
 - L'hôte distant vérifie qu'un seul facteurs et le poste client vérifie un autre facteur (ex: carte à puce avec code PIN ou FIDO avec empreinte)

## Choix du facteur d'authentification

|Facteur|Type de secret|Algorithme de cryptographie utilisé|Lieu du secret|possibilités non exaustives de compromission|Surface d'attaque|authentification forte native|
|------|------|-----|-----|-----|-----|-----|
|facteur de connaissance|Mot de passe|Symetrique (secret partagé)|<ul><li>Poste client</li><li>Serveur authentification</li><li>Element utilisé pour transmettre le secret à l'utilisateur</li><li>Element utilisé pour transmettre le secret au service pour l'enrôlement</li><li>Element utilisé pour transmettre la réinitialisation du mot de passe à l'utilisateur</li><li>Secret potentiellement transmis en brut sur le réseau</li></ul>|<ul><li>Déduction (ex: brute force, mot de passe par défaut, rejeu)</li><li>Hameçonnage</li><li>Ecoute/interception réseau</li><li>vol/perte si l'équipement n'a pas ses données chiffrées</li><li>Récuperation ou utilisation direct sur un équipement compromis qui le connait (admin, messagerie si transmission du secret par courriel, ...), l'utilise ou l'a utilisé (cache)</li><li>Utilisation de la procédure de réinitialisation du mot de passe (mot de passe perdu)</li></ul>|<ul><li>Création du secret: Importante</li><li>Transmission du secret: Importante</li><li>Enrôlement du secret: Importante</li><li>Nombre d'équipement où est le secret lors de l'utilisation: Importante</li><li>Facilité de vol: Importante</li><li>Facilité d'utilisation illégitime depuis poste client compromise: Importante</li></ul>|Non|
|facteur de possession|TOTP|Symetrique (secret partagé avec durée de validité 60sec utilisable plusieurs fois)|<ul><li>Poste client</li><li>Serveur authentification</li><li>Element utilisé pour transmettre le secret à l'utilisateur</li><li>Element utilisé pour transmettre le secret au service pour l'enrôlement</li><li>Secret potentiellement transmis en brut sur le réseau</li><li>Sauvegarde</li></ul>|<ul><li>Hameçonnage</li><li>vol/perte si l'équipement n'a pas ses données chiffrées</li><li>Ecoute/interception réseau</li><li>Récuperation ou utilisation direct sur un équipement compromis qui le connait (admin, messagerie si transmission du secret par courriel, ...), l'utilise ou l'a utilisé (cache</li></ul>|<ul><li>Création du secret: Mineure</li><li>Transmission du secret: Importante</li><li>Enrôlement du secret: Importante</li><li>Nombre d'équipement où est le secret lors de l'utilisation: Importante</li><li>Facilité de vol: Importante</li><li>Facilité d'utilisation illégitime depuis poste client compromise: Importante</li></ul>|Non|
|facteur de possession + facteur de connaissance|calculette OTP avec code PIN|Symetrique (secret partagé avec utilisation unique de chaque code généré)|<ul><li>Poste client</li><li>Serveur authentification</li><li>Element utilisé pour transmettre le secret à l'utilisateur</li><li>Element utilisé pour transmettre le secret au service pour l'enrôlement</li><li>Secret potentiellement transmis en brut sur le réseau</li><li>Sauvegarde</li></ul>|<ul><li>Hameçonnage</li><li>vol/perte si l'équipement n'a pas ses données chiffrées</li><li>Ecoute/interception réseau</li><li>Récuperation ou utilisation direct sur un équipement compromis qui le connait (admin, messagerie si transmission du secret par courriel, ...), l'utilise ou l'a utilisé (cache</li></ul>|<ul><li>Creation du secret: Mineure</li><li>Transmission du secret: Mineure</li><li>Enrôlement du secret: Importante</li><li>Nombre d'équipement où est le secret lors de l'utilisation: Importante</li><li>Facilité de vol: Importante</li><li>Facilité d'utilisation illegitime depuis poste client compromise: Importante</li></ul>|Non|
|facteur de possession|Certificat privé/public hors composant de sécurité|Asymetrique|<ul><li>Poste client</li><li>Element utilisé pour transmettre le secret à l'utilisateur</li><li>PKI ou serveur qui génère les certicats</li><li>Sauvegarde</li></ul>|<ul><li>vol/perte possible si l'équipement n'a pas ses données chiffrées</li><li>Récuperation ou utilisation direct sur un équipement compromis qui possède la clé privée</li></ul>|<ul><li>Creation du secret: Modéré</li><li>Transmission du secret: Importante</li><li>Enrôlement du secret: Mineure</li><li>Nombre d'équipement où est le secret lors de l'utilisation: Mineure</li><li>Facilité de vol: Modéré</li><li>Facilité d'utilisation illégitime depuis poste client compromise: Importante</li></ul>|Oui|
|facteur de possession + facteur de comportemental|Certificat privé/public dans FIDO2 avec facteur comportemental de toucher|Asymetrique|<ul><li>clé FIDO2</li><li>fabriquant de la clé FIDO2</li></ul>|<ul><li>vol/perte</li><li>La dernière solution consiste à être sur le poste du client et d'attendre qu'il crée une connexion authentifié et sécurisé pour lui voler</li></ul>|<ul><li>Creation du secret: Mineure</li><li>Transmission du secret: Mineure</li><li>Enrôlement du secret: Mineure</li><li>Nombre d'équipement où est le secret lors de l'utilisation: Mineure</li><li>Facilité de vol: Modéré</li><li>Facilité d'utilisation illégitime depuis poste client compromise: Mineure</li></ul>|Oui|
|facteur de possession + facteur de connaissance|Certificat privé/public dans carte à puce avec PIN CODE|Asymetrique|<ul><li>carte à puce</li><li>fabriquant carte à puce</li><li>Element utilisé pour transmettre le code PIN à l'utilisateur</li></ul>|<ul><li>vol/perte possible si le code PIN est simple et/ou que la carte n'a pas une conception sécurisée</li><li>Utilisation direct sur le poste client compromis si celui-ci laisse la carte dans le lecteur</li></ul>|<ul><li>Création du secret: Mineure</li><li>Transmission du secret: Modéré</li><li>Enrôlement du secret: Mineure</li><li>Nombre d'équipement où est le secret lors de l'utilisation: Mineure</li><li>Facilité de vol: Mineure</li><li>Facilité d'utilisation illégitime depuis poste client compromise: Importante</li></ul>|Oui|
|facteur de possession + facteur de inhérent|Certificat privé/public dans FIDO2 avec facteur inhérent empreinte digital|Asymetrique|<ul><li>clé FIDO2</li><li>fabriquant de la clé FIDO2</li></ul>|<ul><li>vol/perte possible si l'attaquant à la possibilité d'usurper l'empreinte digital de l'utilisateur et/ou que la clé n'a pas une conception [sécurisée](https://fidoalliance.org/certification/authenticator-certification-levels/)</li><li>La dernière solution consiste à être sur le poste du client et d'attendre qu'il crée une connexion authentifié et sécurisé pour lui voler</li></ul>|<ul><li>Création du secret: Mineure</li><li>Transmission du secret: Mineure</li><li>Enrôlement du secret: Mineure</li><li>Nombre d'équipement où est le secret lors de l'utilisation: Mineure</li><li>Facilité de vol: Mineure</li><li>Facilité d'utilisation illégitime depuis poste client compromise: Mineure</li></ul>|Oui|


Choix du niveau de sécurité d'authentification en fonction du risque en cas de compromission de l'utilisateur :  

|Risque|Criticité Faible|Criticité Moyenne|Criticité Haute|
|------|------|------|------|
|L'utilisateur authentifié à le droit d'accéder à des données sensibles (Confidentialité)|Authentification Niveau Faible|Authentification Niveau Modéré|Authentification Niveau Fort|
|L'utilisateur authentifié à le droit de modifier des données pouvant porter atteinte à la réputation de la structure (Integrité)|Authentification Niveau Faible|Authentification Niveau Modéré|Authentification Niveau Fort|
|L'utilisateur authentifié à le droit de modifier ou supprimer des données pouvant avoir un impact financier (Integrité/Disponibilité)|Authentification Niveau Modéré|Authentification Niveau Modéré|Authentification Niveau Fort|
|Les droits donnés à l'utilisateur sur le service/application peuvent avoir un impact sur les services ou applications du SI (Integrité/Disponibilité)|Authentification Niveau Modéré|Authentification Niveau Fort|Authentification Niveau Fort|
|Les droits donnés à l'utilisateur sur le service/application peuvent avoir un impact sur la sécurité des personnes|Authentification Niveau Modéré|Authentification Niveau Fort|Authentification Niveau Fort|


|Niveau d'authentification|Choix de secret(s) par ordre de préférence|
|------|------|
|Authentification Niveau Faible|<ul><li>Tous les choix possibles</li></ul>|
|Authentification Niveau Modéré|<ul><li>FIDO2 avec facteur inhérent</li><li>Carte à puce avec PIN (enlever carte à puce du lecteur après chaque authentification</li><li>FIDO2 avec facteur comportemental + certificat ou OTP/TOTP</li><li>certificat + OTP/TOTP ou mot de passe</li></ul>|
|Authentification Niveau Fort|<ul><li>FIDO2 avec facteur inhérent</li><li>Carte à puce avec PIN (enlever carte à puce du lecteur après chaque authentification</li><li>FIDO2 avec facteur comportemental + certificat ou OTP/TOTP</li></ul>|


La qualité du secret réside principalement dans le fait qu'il soit basé ou non sur un algorithme de cryptographie.

Si l'on souhaite un niveau de sécurité élevé, on choisira un secret basé sur des algorithme de cryptographie validés par l'[ANSSI](https://www.ssi.gouv.fr/uploads/2021/03/anssi-guide-selection_crypto-1.0.pdf) ou le [NIST](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-131Ar1.pdf).

L'ANSSI dans sa documentation sur l'authentification indique qu'une authentification forte est différente d'une authentification multi-facteur qui peut finalement avoir un niveau faible en fonction des choix.
Le NIST indique lui 3 niveaux de sécurité AAL1 à AAL3 où ils indiquent avec précision les facteurs et les protocoles autorisé pour chaque niveau.

Une authentification sécurisée doit :
 - Utiliser des algorithmes de cryptographie validés par l'[ANSSI](https://www.ssi.gouv.fr/uploads/2021/03/anssi-guide-selection_crypto-1.0.pdf) ou le [NIST](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-131Ar1.pdf) à tous les niveaux : secrets, poignée de main et connexion chiffrée.
 - ne pas être vulnérable l’écoute clandestine ou l'homme du milieu ;
 - ne pas être vulnérable aux attaques par rejeu/replay ;
 - ne pas être vulnérable aux attaques de type phishing ;
 - ne pas être vulnérable à la perte du secret en cas de compromission du serveur d'authentification ;
 - être initialisée par une action du client de préférence comportemental ;
 - limiter la durée de validité avant une réauthentification totale (ex NIST: 12h ou 15 minutes en cas d'inactivité)
 - Tous les composants/équipements doivent respecter une politique de sécurité forte (mises à jour de sécurité, limitation de la surface d'attaque, moindre privilège, ...)

## Architecture d'authentification et d'autorisation

Le choix de l'architecture dépendra de votre contexte et de vos besoins, mais pensez qu'il est toujours préférable de minimiser la surface d'attaque ainsi que la complexité des mécanismes. Choisissez une architecture avec des protocoles que vous maitrisez.

|Architecture|Exemple(s)|Avantage(s)|Inconvénient(s)|schema|
|------|------|------|------|-----|
|Autorisation et Authentification en local|SSH par certificat RSA|<ul><li>la surface d'attaque est limitée au poste client et au service où l'on souhaite accéder</li></ul>|<ul><li>Gestion des authentifications et autorisations dans le temps sur les services du SI. Pour limiter cet inconvénient il est possible d'utiliser un outil de gestion de configuration comme ansible par exemple</li><li>il n'y a pas de rupture protocolaire</li><li>Il est nécessaire de centraliser les logs de tous les services pour avoir les traces des authentifications dans le SI</li></ul>|![image](https://user-images.githubusercontent.com/1132448/179550504-c4a00d37-3642-4c92-a35b-4bd690a0031d.png)|
|Autorisation en local et authentification par délégation|Authentification web par certificat|<ul><li>une gestion centralisée de l’authentification doit être mise en œuvre pour favoriser le suivi des comptes et le respect de la politique de sécurité (ex. : renouvellement des secrets d’authentification, verrouillage ou révocation). Il est primordial que l’entité soit en mesure de réagir rapidement en cas de compromission, suspectée ou avérée, d’un compte d’administration, en bloquant son utilisation sur un maximum de ressources.</li></ul>|<ul><li>la surface d'attaque est plus importante car on peut attaquer le poste client, le service distant ainsi que le serveur de délégation (exemple ici la PKI).</li><li>Selon le type de délégation; il peut être nécessaire de centraliser les logs de tous les services pour avoir les traces des authentification dans le SI (exemple PKI).</li><li>il n'y a pas de rupture protocolaire</li><li>En cas de compromission du serveur de délégation tous les éléments dépendant son perdu, il est donc important de comprendre le mécanisme d'authentification et d'analyser la surface d'attaque de celui-ci et du service.</li></ul>|![image](https://user-images.githubusercontent.com/1132448/179551415-c7507701-ad15-4b73-b112-f27e68b42eae.png)|
|Autorisation et authentification par délégation|AD/kerberos, oauth2, openid, webauthn|<ul><li>une gestion centralisée de l’authentification doit être mise en œuvre pour favoriser le suivi des comptes et le respect de la politique de sécurité (ex. : renouvellement des secrets d’authentification, verrouillage ou révocation). Il est primordial que l’entité soit en mesure de réagir rapidement en cas de compromission, suspectée ou avérée, d’un compte d’administration, en bloquant son utilisation sur un maximum de ressources.</li><li>Le secret n'a pas besoin d'être connu par le service/l'application cible</li></ul>|<ul><li>une fois qu'un token d'accès est utilisé, il n'est pas possible de l'invalider. Celui-ci sera valide durant toute sa durée de validité. Exception, vous pouvez changer le secret de l'application qui cassera tous les tokens (sous Kerberos par exemple c'est clé est KRBGT à changer 2 fois).</li><li>la surface d'attaque est parfois importante car le serveur de délégation peut proposer plusieurs sources d'authentification tiers. Il est donc important de maitriser l'ensemble des composants de l'authentification pour comprendre le risque et la source d'attaque disponible.</li><li>il n'y a pas de rupture protocolaire</li><li>En cas de compromission du serveur de délégation tous les éléments dépendant son perdu, il est donc important de comprendre le mécanisme d'authentification et d'analyser la surface d'attaque de celui-ci et du service</li></ul>|![image](https://user-images.githubusercontent.com/1132448/179553587-e890cd37-d5a7-4816-85da-0505da679ef9.png)|
|Autorisation et authentification par délégation avec rupture protocolaire|Bastion (ex: guacamole)|<ul><li>une gestion centralisée de l’authentification doit être mise en œuvre pour favoriser le suivi des comptes et le respect de la politique de sécurité (ex. : renouvellement des secrets d’authentification, verrouillage ou révocation). Il est primordial que l’entité soit en mesure de réagir rapidement en cas de compromission, suspectée ou avérée, d’un compte d’administration, en bloquant son utilisation sur un maximum de ressources.</li><li>Le secret n'a pas besoin d'être connu par le service/l'application cible</li><li>Il y a une rupture protocolaire avec un point de centralisation qui permet d'identifier et couper toutes les connexions en cours très rapidement</li></ul>|<ul><li>la surface d'attaque est limité car situé du poste d'admin au service d'authentification, le service où l'on souhaite accéder est inaccessible sans passer par le reverse proxy d'authentification qui est en rupture protocolaire. Si l'authentification utilise une ressource tierce alors il faudra analyser la surface d'exploitation ajoutée.</li><li>la surface d'attaque est parfois importante car le serveur de délégation peut proposer plusieurs sources d'authentification tiers. Il est donc important de maitriser l'ensemble des composants de l'authentification pour comprendre le risque et la source d'attaque disponible.</li><li>En cas de compromission du serveur d'authentification et d'autorisation tous les éléments dépendant son perdu, il est donc important de comprendre le mécanisme d'authentification et d'analyser la surface d'attaque de celui-ci et du service</li></ul>|![image](https://user-images.githubusercontent.com/1132448/179555898-080c0e71-b7f4-409a-8916-d7491feb487c.png)|

### Conclusion architecture

La méthode d'authentification est importante à choisir en fonction de ces besoins qu'elle soit au niveau de la sécurité ou de l'exploitation.  
Il est important de ne pas avoir un référentiel d'authentification unique pour tout un SI mais d'essayer à minima de séparer les activités suivantes:
  - Bureautique => on sera souvent sur de l'authentification basée sur un AD
  - Métier => il est important aussi d'avoir du SSO pour l'expérience utilisateur, on privilégiera un référentiel diffèrent et en fonction des données et du besoin de confidentialité on utilisera une authentification adaptée (forte ou non).
  - Administration du SI => l'administration du SI est la base de la sécurité, a partir du moment où le référentiel de cette activité est compromise, tout le SI est compromis. Il est donc important d'utiliser une authentification forte avec une authentification centralisé pour l'administration du SI. L'utilisation de la rupture protocolaire permettra de pouvoir effectuer une surveillance de l'ensemble de l'administration en un point unique, permettant d'avoir une action de remédiation en ce point impactant immédiatement le SI.

Sur de très grand SI, on peut augmenter les zones par la combinaison de plusieurs critères d’homogénéité, parmi lesquels :
 - de criticité métier (ex. : haute, moyenne, basse) ;
 - organisationnels (ex. : administration interne ou infogérée) ;
 - d’exposition (ex. : à Internet, à des fournisseurs, exclusivement interne) ;
 - réglementaires (ex. : données de santé, données personnelles, données relevant du secret de la défense nationale) ;
 - géographiques (ex. : découpage par pays).

### Spécificité Administration du SI

#### Poste d'administration
Il doit faire l’objet d’une sécurisation physique et logicielle afin de restreindre au mieux les risques de compromission.  
Tous les disques doivent être chiffrés.  
Toutes les installations doivent être contrôlées :
 - analyse antivirale, analyse en bac à sable, vérification de signature électronique, traçabilité à l’aide d’un condensat (hash), etc. ;
 - contrôle de la source de téléchargement, de l’émetteur, etc.

Un administrateur peut disposer d’un poste d’administration unique pour l’administration de différentes zones de confiance (hors classifications au sens de l’IGI 1300), aux conditions suivantes :
 - la sécurisation du poste d’administration doit être en phase avec les besoins de sécurité de la zone de confiance administrée la plus exigeante (ex. : un poste d’administration physiquement pour administrer un SI critique peut servir à l’administration d’un SI standard) ;
 - les serveurs outils accessibles depuis le poste d’administration ne doivent pas être mutualisés pour l’administration de deux zones de confiance distinctes (en d’autres termes, un serveur outils reste dédié à une unique zone de confiance et cloisonné dans une zone d’administration) ;
 - les éventuels outils locaux au poste d’administration permettant un accès direct aux ressources administrées doivent être cloisonnés afin d’éviter tout rebond entre deux ressources administrées de deux zones de confiance distinctes à travers le poste d’administration (en d’autres termes, sur un poste d’administration mutualisé, les environnements d’exécution d’outils d’administration locaux de deux zones de confiance distinctes doivent être distincts, par exemple par l’utilisation de la conteneurisation) ;
 - l’accès aux différentes zones de confiance depuis le SI d’administration doit respecter le cloisonnement, physique ou logique, entre zones de confiance (en d’autres termes, deux pares-feux physiques ou un pare-feu configuré avec deux DMZ sont déployés en périphérie du SI d’administration, pour respecter le cloisonnement des zones de confiance).

![image](https://user-images.githubusercontent.com/1132448/177759987-058c004d-2f41-430f-bae5-b154c74f4078.png)

![image](https://user-images.githubusercontent.com/1132448/177760599-777214f7-37f6-446c-bf56-2913306dbe95.png)

#### Zone d'administration
La zone d'administration contient les poste d'administration/outils d'administration, elle est isolée du reste du SI pour limiter les attaques vers cette zone et donc protéger les postes d'administration.
Pour intégrer la zone d'administration privilégier le protocole i802.1x, prise réseau dédié dans une bureau sécurisé, accès VPN IPsec (pour arriver depuis l'externe).

![image](https://user-images.githubusercontent.com/1132448/177767140-cf16be7a-108b-4c49-a60b-28a959ede605.png)

## Spécificité environnement Windows
L'information logon type (contenu dans l'évènement 4624/4625 sur le serveur cible) permet d'identifier le type d'authentification réalisé et le risque d'avoir communiqué les identifiants en clair qui pourront être récupérable par une attaque sur la machine cible et rejoués.  

**Il faut bien comprendre que l'authentification Windows utilise seulement le "facteur de connaissance" avec deux grands protocoles d'authentification "NTLM" et "Kerberos". Seul l'utilisation de Kerberos avec l'utilisation d'algorithmes de cryptographique validés par les directives de l'ANSSI ainsi que l'utilisation des politiques d'authentification (limiter l'utilisation d'identifiant depuis des postes/machines spécifiques vers des serveurs identifiés) peuvent permettre de s'approcher d'une authentification forte (ex: pas d'utilisation d'algorithmes DES ou RC4).**

|Logon  | Type Name     | Type d'authentificateur |   Description | Risque de vol de secrets |
| ------|-----|-----|-----|-----|
|0    |Utilisé par le système||     Utilisé lors du démarrage| Non |
|2    |Interactive |    Password, SmartCard|    Quand un utilisateur se connecte à cette machine par la console | Oui |
|3    |Network |  Password, ticket Kerberos, NT Hash| Quand un utilisateur ou un ordinateur se connecte à cette machine r à partir du réseau.| Non sauf si une délégation est activée sur le compte utilisateur utilisé |
|4    |Batch |    Password | Quand une tache planifiée est lancée par un utilisateur spécifique | Oui |
|5    |Service |  Password | Quand un service est lancé par un utilisateur spécifique | Oui |
|7    |Unlock|    Password | Quand un utilisateur déverrouille sa session | Non|
|8    |NetworkCleartext |     Password| Quand un utilisateur passe ses identifiants en clair au service présent sur cette machine auquel il veut accèder | Oui |
|9    |NewCredentials | Password | Quand un utilisateur utilise la commande "runas" avec l'option "/netonly" pour se connecter à une ressource réseau avec un autre identifiant. | Oui |
|10 |RemoteInteractive |      Password, SmartCard |   Quand un utilisateur se connecte sur cette machine à distance avec RDP | Oui |
|11 |CachedInteractive |      Password|   Quand un utilisateur se connecte à cette machine par la console avec un compte du domaine mais que la machine n'est pas connectée au réseau de l'entreprise. L'authentification utilise les informations dans le cache. | Oui|
|12   |CachedRemoteInteractive|     Password|   Quand un utilisateur se connecte à distance sur cette machine par RDP avec un compte du domaine mais que la machine n'est pas connectée au réseau de l'entreprise. L'authentification utilise les informations dans le cache. | Non|
|13 |CachedUnlock |     Password    |Quand un utilisateur déverrouille sa session mais que la machine n'est pas connectée au réseau de l'entreprise. L'authentification utilise les informations dans le cache. | Non|


| Type d'authentificateur |Intérêt du vol|
| ------|-----|
| Password|L'identifiant est intéressant pour un attaquant seulement si : <ul><li>il est utilisé à d'autre endroit (rejeu de mot de passe)</li><li>le compte permet d'accéder à plusieurs ressources</li></ul>|
| NT hash|Le NT Hash est intéressant pour un attaquant seulement si: <ul><li>son cassage permet d'identifier un mot de passe utilisé à d'autre endroit (rejeu de mot de passe)</li><li>le NT hash permet d'accéder à plusieurs ressources (pass-the-hash)</li></ul>|
| Ticket Kerberos| Le ticket Kerberos est intéressant pour un attaquant seulement si: <ul><li>le ticket Kerberos TGT permet d'accéder à plusieurs ressources (pass-the-ticket)</li></ul> Le vol d'un Ticket TGS n'a pas réellement de sens (car si l'attaquant le vol sur le poste client alors il peut voler directement le mot de passe ou le TGT; et s'il le vol sur la machine cible, cela veut dire qu'il a déjà accès à cette machine avec des privilèges|

| Type d'authentificateur | Élément récupérable par un attaquant selon sa position |
| ------|-----|
| Password| <ul><li>Password sur le poste client</li><li>Password durant la communication si la communication n'est pas sécurisée</li><li>Password sur la machine cible</li></ul> |
| NT hash | <ul><li>Password sur le poste client</li><li>HASH NT sur le poste client</li><li>Defi/réponse NTLM protocole (utilisé en version 1 ou 2 lors de l'utilisation de NT hash) si la communication n'est pas sécurisé (ce qui dans le protocole de base NTLM n'est pas le cas) qui permettra differentes attaques (relai/brute force/..)</li><li>Defi/réponse NTLM protocole (utilisé en version 1 ou 2 lors de l'utilisation de NT hash) transmis à la machine cible qui permettra différentes attaques (relai/brute force/..)</li></ul> |
| Ticket kerberos|<ul><li>Password sur le poste client</li><li>HASH NT sur le poste client</li><li>Ticket Kerberos TGT sur le poste client</li><li>Ticket kerberos TGT sur le serveur cible si une délégation est activée</li></ul>|


|Services/outils | logon type | Exemple | Risque de vol de secrets |
| ------|-----|-----|-----|
|Accès au partage de fichier |3|commande "net use"| Non sauf si une délégation est activée sur le compte utilisateur utilisé|
|WMI à distance   |3|command "wmic /node"| Non sauf si une délégation est activée sur le compte utilisateur utilisé|
|WinRM ou powershell à distance     |3|" Invoke-command -ComputerName "machinedistante" -filepath C:\script.ps1"| Non sauf si une délégation est activée sur le compte utilisateur utilisé|
|integrated windows authentification (IWA)[https://docs.microsoft.com/fr-fr/aspnet/web-api/overview/security/integrated-windows-authentication]       |3|Utilisé sur les services web où l'utilisateur est connecté sans demande d'identifiant| Non sauf si une délégation est activée sur le compte utilisateur utilisé|
|Share Admin 1er cas|3|"psexec" avec l'utilisateur connecté| Non sauf si une délégation est activée sur le compte utilisateur utilisé|
|Share Admin 2eme cas|3 puis 2|"psexec -u" avec un identifiant different de l'utilisateur connecté| Oui|
|Creation d'une tache planifiée avec un utilisateur spécifique|4|Creation d'une tache planifiée pour la sauvegarde sur un serveur avec un identifiant domaine afin qui puisse transmettre la sauvegarde sur un partage du SI| Oui|
|Creation d'un service lancé par un utilisateur spécifique (diffèrent du système local)|5|Creation d'un service avec un utilisateur du domaine afin qu'il puisse accéder à certaines ressources du SI| Oui|
|SSH|8|ssh qui utilise une authentication login/password basé sur l'AD ou compte windows local| Oui|
|FTP|8|FTP qui utilise une authentication login/password basé sur l'AD ou compte windows local| Oui|
|IIS|8|IIS qui utilise une authentication login/password basé sur l'AD ou compte windows local (hors IWA qui ne demande pas de login/passwd)| Oui|
|Webmail exchange OWA|8|OWA qui utilise une authentication login/password basé sur l'AD ou compte windows local (hors IWA qui ne demande pas de login/passwd)| Oui|
|Sharepoint|8|Sharepoint qui utilise une authentication login/password basé sur l'AD ou compte windows local (hors IWA qui ne demande pas de login/passwd)| Oui|
|Sharepoint|8|Sharepoint qui utilise une authentication login/password basé sur l'AD ou compte windows local (hors IWA qui ne demande pas de login/passwd)| Oui|
|MS Exchange ActiveSync|8|MS Exchange ActiveSync qui utilise une authentication login/password basé sur l'AD| Oui|
|RunAs|9|Commande "runas /netonly"|Oui|
|RDP  |10|connexion avec le bureau à distance vers une machine à administrer avec des identifiants du domaine| Oui|
|RDP avec option NLA    |3 puis 2|connexion avec le bureau à distance vers une machine à administrer avec des identifiants du domaine| Oui|
|RDP avec option [restrictedadmin](https://docs.microsoft.com/fr-fr/windows/security/identity-protection/remote-credential-guard)   |10|connexion avec le bureau à distance vers une machine à administrer avec des identifians du domaine| Non car l'authentification utilise le compte local, attention après a ne pas effectuer une authentification de type logon2 avec un compte à privilege du domaine|
|RDP avec option [Remote credential guard](https://docs.microsoft.com/fr-fr/windows/security/identity-protection/remote-credential-guard) |10|connexion avec le bureau à distance vers une machine à administrer avec des identifiants du domaine| Oui mais seulement durant la période de connexion en RDP, dès que la connexion est terminée alors tous les secrets sont effacés du serveur|
|?|?|?|Si vous avez un doute sur un outils ou un service en termes d'utilisation de logon type, alors réaliser un vérification en amont avec un compte sans privilège sur le serveur cible et rechercher dans les journaux d'évènements l'id 4624 afin d'identifier le ou les logon type réalisés.|


|Stockage des identifiants | Description |Secrets récupérables| période de récupération des mots de passe possible |
| ------|-----|-----|-----|
|base [SAM](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#security-accounts-manager-database) (Security Accounts Manager)|Il s'agit de la base des utilisateurs du domaine présente sur les contrôleurs de domaine|[NT hash](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#nt-hash)|Sans limite de durée|
|[Active Directory Domain Services](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#adds-database-ntdsdit) (AD DS) database|Il s'agit de la base des utilisateurs du domaine présente sur les contrôleurs de domaine|<ul><li>[NT hash](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#nt-hash)</li><li>[NT hashes password History](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#nt-hash) selon la configuration</li><li>[LM hash](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#lm-hash) selon la configuration et la version du controleur de domaine</li></ul>|Sans limite de durée|
|[LSASS](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#lsass-process-memory) (Local Security Authority Subsystem Service) process memory|Il s'agit du processus qui stock les secrets en mémoire utilisés sur les sessions windows actives|<ul><li>[Reversibly encrypted plaintext](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#plaintext-credentials)</li><li>Kerberos tickets (TGTs, service tickets)</li><li>[NT hash](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#nt-hash)</li><li>[LM hash](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#lm-hash)</li></ul>|durée de vie du processus|
|[LSA](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#lsa-secrets-on-the-hard-disk-drive) (Local Security Authority)|Il s'agit d'une base sur disque qui garde les identifiants liés à certaines activités du système à partir de comptes non local||<ul><li>Account password for the computer’s AD DS account</li><li>Account passwords for Windows services that are configured on the computer</li><li>Account passwords for configured scheduled tasks</li><li>Account passwords for IIS application pools and websites</li></ul>|Sans limite de durée, jusqu'a la fin de l'utilisation du compte|
|[Mscache](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#windows-logon-cached-password-verifiers)|Il s'agit du système windows qui permet de garder en cache les identifiants de connexion pour un domaine afin que si celui-ci n'est pas joignable alors il reste possible de s'authentifier localement grâce à ce cache.|Domain Cache credential([DCC1 ou DCC2](https://blog.gentilkiwi.com/securite/mscache-v2-dcc2-iteration) => il s'agit du hash ou du mot de passe chiffré en utilisant pour sel le nom de l'utilisateur). Pour l'utiliser on casse le chiffrement pour récupérer l'information.|Garde les 10 derniers hash de connexion sans limite de temps|
|[Credential Manager store](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)#credential-manager-store)|Il s'agit du gestionnaire de mot de passe windows "Windows Vault" qui stock les mots de passe utilisés par l'utilisateur sur des sites web, applications, ...|plaintext password|Sans limite de durée|


**Bonnes pratiques d'administration windows pour éviter le vol de ses secrets :**

 - Pas de compte de domaine à privilège avec la délégation d'activé
 - Un compte d'administration ne doit être utilisé que sur le poste de l'administrateur (poste durci en matière de sécurité : politique d'exécution, politique firewall local, chiffrement des disques, pas d'accès bureautique pour limiter la surface d'attaque, ...). La compromission d'un poste d'administration peu importe le type d'authentification utilisé (carte à puce, fido, ...) et les protections mises en place sur le SI (bastion, politique d'authentification, segmentation, ...) rend "compromettable" toutes les ressources administrées par ce poste (niveau, ex: tiers 1). Dans le même principe, un contrôleur de domaine compromis rend "compromettable" toutes les ressources utilisant ce moyen d'authentification (tous les niveaux tiers0/1/2 - sauvegarde, hyperviseur, machine, service, ...).
 - Un compte d'administration ne doit jamais être utilisé en dehors du cadre d'administration. Concrètement, il ne doit jamais être utilisé pour accéder à des services comme sharepoint, webmail owa, ftp, ssh, ou tout autre protocole qui utilise un logon type diffèrent de 3.
 - Lors de l'administration seul les authentifications "forte" seront autorisées pour cela le compte sera placé dans le groupe ["utilisateur protégé"](https://docs.microsoft.com/fr-fr/windows-server/security/credentials-protection-and-management/protected-users-security-group)
 - Si dans le cadre de l'administration vous utilisez des outils qui n'utilise pas l'authentification windows comme par exemple VNC alors il faut impérativement utiliser un mécanisme [d'authentification forte dans le sens de l'ANSSI](https://www.ssi.gouv.fr/uploads/2021/10/anssi-guide-authentification_multifacteur_et_mots_de_passe.pdf#section.2.5)
 - Pas de connexion en local/physique sur une machine avec un compte à privilège de domaine
 - Utilisation exclusivement d'outils d'administration à distance utilisant du logon type 3 sauf pour les exceptions suivantes :
   - Connexion possible avec le compte à privilège d'admin domaine (peu importe le logon type) seulement sur les contrôleurs de domaine (si un attaquant est sur le DC alors toute la sécurité est déjà en échec).
   - Privilégier les comptes locaux à la machine à administrer pour n'importe quel type de logon à condition que le mot de passe du compte local ne soit pas réutilisé ailleurs ou prévisible (ex: mot de passe == nom de la machine).
 - Si vous utilisez un Bastion qui se connecte sur des machines windows, vérifiez que son utilisation est conforme à ces préconisations. Il est possible que le bastion ne soit pas en capacité de prendre un charge un utilisateur présent dans le groupe ["utilisateur protégé"](https://docs.microsoft.com/fr-fr/windows-server/security/credentials-protection-and-management/protected-users-security-group).
 - Si vous avez un doute sur un outils ou un service en termes d'utilisation de logon type, alors réaliser une vérification en amont avec un compte sans privilège sur le serveur cible et rechercher dans les journaux d'évènements l'id 4624 afin d'identifier le ou les logon type réalisés.
 - Mettez en place des stratégies d'authentification afin de limiter l'authentification aux comptes à privilèges exclusivement depuis les postes d'administration (défense en profondeur), ce qui permettra en cas du non-respect de certaines mesures ci-dessus et donc la récupération d'un secret par un attaquant (sur une machine administrée) de ne pas pouvoir l'utiliser si celui-ci n'a pas un accès à votre poste d'administration.

**Point d'attention :** il faut se rappeler que l'authentification sous windows repose sur l'unique facteur de connaissance. De plus, la surface d'attaque d'un contrôleur de domaine est souvent très importante (DNS, DHCP, AD, Partage de fichier sysvol, PKI, parfois spooler, ...). De ce fait, il peut être judicieux de ne pas baser l'authentification (en particulier des canaux d'administration) de tout le SI sur un AD mais seulement la partie bureautique. Si un service utilise l'authentification AD pour accéder à un service web par exemple, alors la compromission de l'AD n'aura pas un impact sur l'intégrité et la disponibilité du service à condition que la machine qui l'héberge ne soit pas dans l'AD (car au delà du service d'authentification que procure un AD, il faut bien comprendre qu'il s'agit d'un gestionnaire de configuration pour toutes les machines enrôlées dans le domaine).

Références utilisées :
 - https://docs.microsoft.com/fr-fr/windows-server/identity/securing-privileged-access/reference-tools-logon-types
 - https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v%3Dws.11)
 - https://techgenix.com/Logon-Types/
 - https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them
 - https://www.sstic.org/2017/presentation/administration_en_silo/
 - https://www.sstic.org/2020/presentation/analyse_de_la_scurit_rdp__nla_quel_apport_pour_votre_scurit_/

## Références
 - https://www.ssi.gouv.fr/uploads/2021/10/anssi-guide-authentification_multifacteur_et_mots_de_passe.pdf
 - https://www.ssi.gouv.fr/uploads/2018/04/anssi-guide-admin_securisee_si_v3-0.pdf
