# Mise en oeuvre de Traefik Proxy et sécurisation de l'accès à l'application avec BasicAuth HTTP, filtrage d’adresse IP et authentification OAuth (Google ID)

Ce projet vise à déployer une application de vote sur un cluster AKS, tout en implémentant des couches de sécurité avancées telles que l’authentification, le filtrage d’adresses IP, le chiffrement TLS et l’intégration OAuth.

## Objectif

- Déployer une application sur un cluster Kubernetes (AKS).
- Configurer un contrôleur d’entrée (Traefik) pour gérer le routage et la sécurité.
- Rendre l’application accessible via un domaine personnalisé sécurisé.

## Étapes Principales

### 1. Déploiement du Cluster AKS

- Création d’un cluster AKS avec un nœud (peut être étendu pour des besoins de production).
- Configuration des ressources nécessaires pour héberger l'application Kubernetes, incluant les services, pods, et secrets.

### 2. Installation de Traefik

- Déploiement de Traefik en tant que contrôleur d'entrée sur le cluster.
- Configuration d’un service LoadBalancer pour obtenir une IP publique.
- Vérification que Traefik est opérationnel et prêt à router les requêtes.

### 3. Configuration DNS

- Obtention d’un domaine ou sous-domaine via un fournisseur DNS.
- Création d’un enregistrement A pointant vers l’adresse IP publique du service LoadBalancer (Traefik).

### 4. Déploiement de l’Application de Vote

- Création des fichiers de configuration YAML pour les ressources Kubernetes suivantes :
    - Déploiement : pour définir les pods contenant l’application.
    - Service : pour exposer l’application au sein du cluster.
    - Ingress : pour connecter Traefik à l’application.
- Vérification que l’application est fonctionnelle à l’adresse configurée.

### 5. Authentification HTTP Basic

- Génération des identifiants utilisateur/mot de passe sous forme cryptée (via htpasswd).
- Création d’un secret Kubernetes pour stocker ces identifiants de manière sécurisée.
- Configuration d’un middleware dans Traefik pour activer l’authentification HTTP Basic.

### 6. Filtrage d’Adresse IP

- Ajout d’une politique de filtrage dans le middleware de Traefik.
- Restriction de l’accès à l’application uniquement aux plages d’adresses IP définies dans une liste blanche.

### 7. Sécurisation avec Certificat TLS

- Génération de certificats TLS via Let’s Encrypt ou une autre autorité de certification.
- Intégration de ces certificats dans Traefik pour sécuriser le trafic HTTP avec HTTPS.
- Configuration de la redirection automatique du trafic HTTP vers HTTPS.

### 8. Authentification TLS Client

- Création d’un certificat client pour chaque utilisateur autorisé.
- Ajout des certificats clients dans une liste approuvée (CA) pour permettre leur validation par Traefik.
- Mise en place de la validation côté serveur pour refuser les connexions non authentifiées.

### 9. Intégration OAuth avec Google

- Création d’un projet dans la console Google Cloud pour activer OAuth 2.0.
- Récupération des identifiants client et secret nécessaires pour l’intégration.
- Configuration d’un middleware Traefik pour rediriger les utilisateurs vers la page de connexion Google.
- Vérification que seuls les utilisateurs authentifiés peuvent accéder à l’application.

## Résultat Attendu

À la fin du projet, l'application est :

- Accessible via un domaine personnalisé (par exemple, vote.example.com).
- Sécurisée grâce au chiffrement HTTPS, au filtrage d'adresses IP et à l’authentification (HTTP Basic, certificats TLS ou OAuth).
- Facile à gérer grâce à Traefik et aux ressources Kubernetes définies.

## Technologies Utilisées

- Kubernetes (AKS) : Gestion des conteneurs et orchestration.
- Traefik : Contrôleur d’entrée pour le routage et la sécurité.
- OAuth (Google) : Authentification utilisateur.
- DNS : Configuration du domaine personnalisé.
- Certificats TLS : Sécurisation du trafic réseau.
- YAML : Fichiers de configuration pour Kubernetes.

## Pré-requis
- Compte Azure pour créer un cluster AKS.
- Un domaine ou sous-domaine configuré.
- Accès à un outil de gestion Kubernetes (kubectl).
- Traefik installé comme contrôleur d’entrée.
