---
title: Brief11 - Presentation
tags: presentation
slideOptions:
  theme: moon
  transition: 'fade'
  spotlight:
    enabled: true
---

# Traefik Proxy : sécuriser l'accès à l'application

---

### Authentification BasicAuth HTTP

![](https://hackmd.io/_uploads/SkgAPY4F2.png)

----

### Avantages

- Simple
- Polyvalent/Compatibilité
- Rapide et efficace
- Encodé donc confidentiel

----

### Inconvénients

- Sécurité non robuste
- Stockage des informations d'identification
- Pas de gestion des droits d'accès
- Déconnexion difficile

=> Adapté pour l'utilisation interne ou pour des situations où la sécurité n'est pas une préoccupation majeure

---

### Filtrage d’adresse IP

Adresse IP unique acceptée par le site
![](https://hackmd.io/_uploads/SkPD_YVFn.png =700x)

----

### Avantages

- Sécurité : autorise uniquement les adresses IP spécifiques
- Contrôle d'accès possible (utilisateur, réseaux, zones)
- Réduction du trafic indésirable : meilleure performance

----

### Inconvénients

- Falsification des adresses IP possible : usurpation, VPN, proxy
- Difficulté de gestion : maintenance compliquée
- Impact sur les utilisateurs légitimes si le filtrage est restrictif ou mal configuré

=> A combiner avec d'autres mesures de sécurité

---

### Certificat TLS avec redirection en https

Certifié par Let's Encrypt avec redirection en https
![](https://hackmd.io/_uploads/S1QvYYNFh.png)

----

### Avantages

- Confidentialité des données : chiffrées entre utilisateur et app web
- Intégrité des données garantie par le certificat (protège contre les attaques de manipulation des données)
- Confiance des utilisateurs : renforce la crédibilité
- Amélioration du référencement par les moteurs de recherche

----

### Inconvénients

- Configuration et maintenance : peuvent être compliquées, renouvellement possiblement à refaire, configuration à vérifier périodiquement
- Impact léger sur les performances : charge de traitement, temps de latence
- Redirection incorrecte en cas de problèmes de configuration

=> Si configuration soignée et bonne maintenance, avantageux

---

### Authentification avec certificats TLS client

Test sans certificat client : bad certificate
![](https://hackmd.io/_uploads/Hkc99KEF2. =600x)

----

### Avantages

- Sécurité renforcée : paire de clés cryptographiques
- Authentification mutuelle entre le client et le serveur
- Gestion centralisée des certificats au sein d'une infrastructure à clé publique (PKI) : facilite la gestion, la révocation et les droits d'accès des clients
- Moins de mots de passe à gérer

----

### Inconvénients

- Complexité de configuration
- Plus difficile pour les utilisateurs finaux : doivent générer, installer et gérer leurs certificats numériques
- Gestion des certificats révoqués : nécessite une surveillance et une gestion continue en cas de compromission

=> Plus adapté quand la sécurité est une priorité et que la gestion des certificats peut être justifiée car plus compliqué. La sécurité peut être renforcée avec d'autres méthodes d'authentification.

---

### Authentification OAuth avec Google ID

Test avec gmail autorisé fonctionne
![](https://hackmd.io/_uploads/By78sFEY2.png =800x)

----

### Avantages

- Simplification pour les utilisateurs
- Sécurité renforcée : l'application reçoit un jeton d'accès limité pour réduire les risques de divulgation
- Authentification à deux facteurs si activé pour le compte Google

----

### Inconvénients

- Dépendance à Google (compte, disponibilité du service)
- Interface utilisateur standardisée
- L'application peut demander des données à l'utilisateur : il doit faire confiance à l'application pour gérer ses données

---

## Difficultés rencontrées

- Beaucoup de recherches/documentation
- Difficultés pour savoir comment organiser les fichiers yaml
