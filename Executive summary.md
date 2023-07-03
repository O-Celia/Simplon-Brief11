# Brief 11 : Mise en œuvre de Traefik Proxy pour sécuriser l'accès à l'application Voting App

## Chapitre 1 : Déployer un cluster AKS
    
### Créer un cluster AKS avec 1 node

Connexion azure CLI
``az login``

Création d'un groupe de ressource
``az group create --name B11Celia --location westeurope``

Création du cluster AKS avec 1 nodes et la clé SSH de l'utilisateur dans le groupe de ressource
``az aks create -g B11Celia -n AKSCluster --node-count 1 --ssh-key-value ./.ssh/id_rsa.pub --enable-managed-identity -a ingress-appgw --appgw-name B11Gateway --appgw-subnet-cidr "10.225.0.0/16"``

Téléchargement des informations d’identification et configuration de l’interface de ligne de commande Kubernetes
``az aks get-credentials --resource-group B11Celia --name AKSCluster``
