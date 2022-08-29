# Methodes de détection d'evenement suspects
## VPN
 - Utilisateur: Horaires suspects
 - Utilisateur: nombre de connexions totales
 - Utilisateur: nombre d'adresses IP différentes
 - Utilisateur: connexions simultanées
 - Utilisateur: date première connexion et date dernière connexion, équivalent à un utilisateur vient de se connecter et on ne l'a jamais vu avant
 - IP: Nombre d'utilisateurs différents sur une même IP
 - IP: Nombre de connexions totales sur une même IP
 - IP: Connexions simultanées
 - IP: liste noire
 - IP: pays d'origine
 - IP: date première connexion et date dernière connexion, équivalent à une adresse IP vient de se connecter et on ne l'a jamais vu avant

## Proxy sortant internet
 - URL: liste noire
 - URL: utilisation d'une adresse IP sans résolution (ex: http://1.1.1.1/)
 - URL: utilisation d'un port suspect (ex: http://1.1.1.1:666/)
 - URL: mime-type suspect
 - URL: ne repond plus maintenant (plusieurs heures/jours) après la requete initiale
 - Domaine: liste noire
 - Domaine: whois indique date de creation récente
 - Domaine: resolution (reverse, history, pas de NS, pas de MX)
 - Domaine: nombre de clients qui utilisent ce domaine
 - Domaine: date première connexion et date dernière connexion sur ce domaine
 - IP: liste noire
 - IP: nombre de clients qui utilisent cette adresse IP
 - IP: date première connexion et date dernière connexion sur cette adresse IP
 - Client: nombre d'adresses IP et domaines differents contactés
 - Client: date première demande proxy et date dernière demande proxy
 - Client: nombre de connexion simultannée très importante sur une courte periode
 - Client: durée de connexion anormalment longue
 - Client: quantité transmise très importante (upload)
 - Client: user-agent suspect (liste noire)
 - Client: Certificat TLS/SSL client - JA3 (jamais vu avant)
 - Certificat site: autosigné, faux SSL/TLS (ex: tunnel SSH), nom suspect, hebergeur à risque, J3AS (jamais vu avant), pas de NS sur le nom de domaine du certif

## Système d'exploitation windows
 - Autorunsc: nouveau autorun
 - Configuration: changement de la configuration de l'antivirus
 - Configuration: changement de la configuration de la politique d'execution
 - Configuration: changement de la configuration du firewall local
 - Configuration: changement de la configuration de service d'administration (admin share, wmi, winrm, ...)
 - Configuration: changement de la configuration de sécurité
 - Configuration: changement de la configuration réseau (wpad, lan, wifi, ...)
 - Utilisateur: ajout/modification d'un utilisateur local
 - Groupe: ajout d'un utilisateur dans un groupe
 - Utilisateur: (4624) connexion d'un utilisateur qui n'avait encore jamais realisé de connexion sur cette machine (ou pas depuis très longtemps)
 - Service: ajout ou modification d'un service
 - Taches planifiées: ajout ou modification d'une taches planifiées
 - Connexion distante entrante: connexion d'un utilisateur avec une adresse IP qui n'avait encore jamais été vu sur cette machine (couple IP-USER)
 - Processus: (sysmon 1 ou security 4688) chemin complet ou hash jamais vu avant par rapport à l'utilisateur qui lance le processus
 - Processus: (sysmon 1 ou security 4688) ligne de commande suspecte (longueur, contenu hex/base64, emplacement fichier, mots clés)
 - Processus: (sysmon 3 ou sysmon 22) connexion sortante pour le processus jamais vu avant
 - Hardware: ajout d'un peripherique (amovible ou non)
 - Registre: Document office mis en confiance (```HKEY_CURRENT_USER\\Software\\Microsoft\\Office\\**\\Excel\\**\\Trusted Documents\\TrustRecords```)

## Système d'exploitation linux
 - Autorun: nouveau autorun (initd, systemd, container, vm, ...)
 - Utilisateur: ajout/modification d'un utilisateur local
 - Configuration: changement de la configuration du firewall local
 - Configuration: changement de la configuration de service d'administration (ssh, sudo, auth, ...)
 - Configuration: changement de la configuration de sécurité (selinux, auditd, ...)
 - Configuration: changement de la configuration réseau (wpad, lan, wifi, ...)
 - Configuration: changement de la configuration dans "/etc"
 - Utilisateur: connexion d'un utilisateur qui n'avait encore jamais realisé de connexion sur cette machine (ou pas depuis très longtemps)
 - Groupe: ajout d'un utilisateur dans un groupe
 - Connexion distante entrante: connexion d'un utilisateur avec une adresse IP qui n'avait encore jamais été vu sur cette machine (couple IP-USER)
 - Processus: (auditd) chemin complet ou hash jamais vu avant par rapport à l'utilisateur qui lance le processus
 - Processus: (auditd) ligne de commande suspecte (longueur, emplacement fichier, mots clés)
 - Processus: (auditd) connexion sortante pour le processus jamais vu avant
 - Hardware: ajout d'un peripherique (amovible ou non)
 - Kernel: chargement ou modification d'un module (lsmod)
 - Crontab: ajout/modification crontab ou script/executable lancé par crontab
 - Fichier: présence d'un fichier executable static
 - Fichier: changement SGID/SUID/capabilities
 - Fichier: (auditd) erreur d'accès à un fichier
 - Package: erreur d'integrité

## Equipements réseau (firewall, routeur, bastion...)
 - Configuration: changement de la configuration de l'équipement
 - Utilisateur: connexion d'un utilisateur qui n'avait encore jamais realisé de connexion sur cette machine (ou pas depuis très longtemps)
 - Utilisateur: ajout/modification d'un utilisateur
 - Connexion distante entrante: connexion d'un utilisateur avec une adresse IP qui n'avait encore jamais été vu sur cette machine (couple IP-USER)

