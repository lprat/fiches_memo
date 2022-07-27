# Procédure sur compromission d'une boite courriel
## Confinement
Nous vous conseillons d'appliquer les actions de confinement suivantes :
  1. Désactiver le compte des utilisateurs compromis
  2. Couper toutes les sessions des utilisateurs compromis en cours sur le webmail/imap/pop (sur office365 le jeton/token d'actualisation à une durée de vie de 90 jours et peut rester valide après la réinitialisation du mot de passe (https://docs.microsoft.com/fr-fr/azure/active-directory/develop/refresh-tokens), il faut l'invalider par commande powershell: https://docs.microsoft.com/fr-fr/powershell/module/azuread/revoke-azureaduserallrefreshtoken?view=azureadps-2.0) ;
  3. Si l'utilisateur a un accès VPN alors désactiver le compte puis vérifier qu'il n'y a pas de connexions suspectes, sinon couper la session et prévenir immédiatement le CERT Santé ;
  4. Interroger l'utilisateur afin d'identifier s'il s'agit d'un phishing (l'utilisateur a donné son mot de passe à son insu). Si c'est le cas, demander si l'action a été réalisée lorsqu'il était au travail ainsi que l’heure approximative afin d'identifier le lien malveillant pour le bloquer (firewall ou proxy) et vérifier qu'aucune autre personne ne l'a utilisé pour transmettre son mot de passe.
  5. Si l'utilisateur indique ne pas avoir transmis son mot de passe (ou s'il y a un doute):
      * Isoler son poste de travail qui pourrait avoir été compromis. Si l'utilisateur est administrateur local du poste alors il faudra réinitialiser le mot de passe de tous les utilisateurs ayant réalisé une connexion "interactive" (ex: rdp/connexion physique sur le poste) sur le poste et vérifier les connexions VPN pour ces utilisateurs. Prévenez le CERT Santé qui pourra vous aider dans l'analyse du poste (pensez à extraire tous les logs des connexions sortantes vers internet pour aider à l'analyse) ;
      * Confirmer avez l'utilisateur, qu'il n'utilise pas un poste "personnel" pour relever sa boite courriel professionnelle (ce qui pourrait être aussi l'origine de la compromission du mot de passe) ;
      * Si l'utilisateur possède un compte google connecté à son navigateur (chrome), celui-ci peut stocker les mots de passe des sites web visités. La compromission du compte google peut être l'origine de la compromission d'un accès externe comme le webmail. Afin d'identifier les mots de passe connus par google, l'utilisateur peut utiliser l'URL suivante: : https://passwords.google.com/
      * Si l'utilisateur possède un compte icloud keychain, celui-ci peut stocker les mots de passe des sites web visités. La compromission du compte icloud peut être l'origine de la compromission d'un accès externe comme le webmail. Afin d'identifier les mots de passe connus par icloud, l'utilisateur peut utiliser l'application "Keychain Access app" pour identifier les mots de passe présents.
  6. Si le serveur de messagerie est hébergé en local (ex: exchange "on premise"), vérifier que le serveur est à jour des derniers patchs de sécurité, si ce n'est pas le cas, passer l'outil MSERT (https://docs.microsoft.com/en-us/microsoft-365/security/intelligence/safety-scanner-download?view=o365-worldwide) sur tous les serveurs du cluster de messagerie. Si MSERT détecte un élément, alors isoler votre serveur et contacter le CERT Santé immédiatement. Dans le cas contraire appliquer les mises à jour à l'ensemble du cluster en débutant par le serveur qui a le rôle de "webmail".
  7. Si l'utilisateur avait des identifiants permettant l'accès à des ressources sur internet dans les courriels présents dans la boite, il est impératif de changer les mots de passe ;
  8. Si l'utilisateur utilisait le mot de passe qui a été volé sur d'autres accès internet personnels ou professionnels, alors il faut changer le mot de passe sur l'ensemble de ces accès ;

## Remediation
Nous vous conseillons d'appliquer les actions de remédiation suivantes :
  1. Vérifier la configuration de la boite de l'utilisateur (sur office365 vous pouvez utiliser l'outil https://github.com/CrowdStrike/CRT) afin de vérifier qu'il n'y a pas eu de modifications illégitimes ; exemple : redirection, ...)
  2. Réinitialiser les mots de passe des utilisateurs compromis, et si possible activer l'authentification multi facteurs (attention il n'est pas possible de mettre en place de l'authentification multi facteurs sur des protocoles dits "legacy" [Imap, acticvesync, smtp, pop, ...], il est donc préférable de ne jamais utiliser ces protocoles sur internet).
  3. Si possible, mettre le "webmail" derrière le VPN. Sinon avec exchange "on premise", filtrer l'accès aux adresses IPs Françaises. Attention, l'activation de cette option, ne permettra plus de voir rapidement si une boite a été compromise par une règle de détection simple (connexion réussie depuis une adresse IP à l'étranger == boîte compromise), cependant, si vous n'avez pas les capacités pour réaliser cette détection en temps réel, il est préférable d'activer cette règle. Dans le cadre d'office 365, activer l'accès conditionnel (https://docs.microsoft.com/fr-ca/azure/active-directory/conditional-access/overview).
  4. Sensibiliser l'utilisateur à ne pas communiquer ses identifiants ou ouvrir des pièces jointes suspectes ;
  5. Identifier pourquoi le courriel de phishing a réussi à contourner vos protections de messagerie et renforcer vos protections. Dans cet objectif, nous vous préconisons de prendre connaissance du document suivant et d'appliquer ses recommandations (https://github.com/cybersante/mx_sec_conf).
  6. Vérifier la durée de rétention des logs (3/6 mois de préférence) sur les éléments suivants : logs des connexions sortantes vers internet, logs des authentifications réussies sur le VPN, logs des requêtes vers votre serveur webmail.
  7. Signaler les courriels malveillants à www.signal-spam.fr ;

## Collecte/Transmission
Elements à transmettre :
  - copie du courriel malveillant reçu par le/les utilisateur(s) dans le cadre d'un signalement pour réception de courriel malveillant ;
  - copie d'un courriel malveillant transmis vers internet/interne dans le cadre d'une compromission d'une boîte avec envoi de courriels illégitimes ;
  - Indiquer les actions de confinement réalisées selon la liste ci-dessus
  - Indiquer les actions de remédiation réalisées selon la liste ci-dessus
