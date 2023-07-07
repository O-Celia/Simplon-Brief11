# Brief 11 : Mise en œuvre de Traefik Proxy pour sécuriser l'accès à l'application Voting App

## Chapitre 1 : Déployer un cluster AKS
    
### Créer un cluster AKS avec 1 node

Connexion azure CLI
``az login``

Création d'un groupe de ressource
``az group create --name B11Celia --location westeurope``

Création du cluster AKS avec 1 nodes et la clé SSH de l'utilisateur dans le groupe de ressource
``az aks create -g B11Celia -n AKSCluster --node-count 1 --ssh-key-value ./.ssh/id_rsa.pub --enable-managed-identity``

## Chapitre 2 : Installation de Traefik

Téléchargement des informations d’identification et configuration de l’interface de ligne de commande Kubernetes
``az aks get-credentials --resource-group B11Celia --name AKSCluster``

Ajouter le référentiel Helm de Traefik aux repositories
``helm repo add traefik https://helm.traefik.io/traefik``
``helm repo update``

Déployer Traefik avec Helm
``helm install traefik traefik/traefik``

Verifier que le pod traefik est bien "Running"
``kubectl get pods``

## Chapitre 3 : Création de DNS pour l'application de vote

Création d’un enregistrement DNS pointant vers l’adresse IP de l’application de vote (cf Services and Ingresses dans AKSCluster => traefik => Ip externe) sur Gandi : http://voteapp.simplon-celia.space

## Chapitre 4 : Déploiement de l'application de vote

Utilisation des fichiers yaml du Brief 10, avec modification du fichier ingress.yaml
```consol
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vote-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: vote.simplon-celia.space
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: votecluster
            port:
              number: 80
```

## Chapitre 5 : Mettre en place une authentification BasicAuth HTTP avec mot de passe local dans la config de Traefik

Installation de apache-utils pour avoir accès à htpasswd
``sudo apt install apache2-utils``

Hasher le mot de passe + base 64
``htpasswd -nb celia password! | openssl base64``

Création d'un secret et d'un mot de passe avec middleware pour lier à traefik
```consol
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: auth
spec:
  basicAuth:
    removeHeader: false
    secret: authsecret
---
apiVersion: v1
kind: Secret
metadata:
  name: authsecret
data:
  users:
    Y2VsaWE6JGFwcjEkLi9QemgwV3YkTVhtQ2lHNldqQ1dYRjMubjlLcWlwLwoK
```

Modification du fichier ingress.yaml : ajout de "router.middlewares", avec namespace-nomMiddleware@kubernetescrd
```consol
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vote-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/router.middlewares: default-auth@kubernetescrd
spec:
  rules:
  - host: voteapp.simplon-celia.space
    http:
      paths:
      - pathType: ImplementationSpecific
        path: /
        backend:
          service:
            name: votecluster
            port:
              number: 80
```

Test de https://vote.simplon-celia.space/ : Demande bien de se connecter

## Chapitre 6 : Mettre en place un filtrage d’adresse IP sur Traefik

Nouveau middleware pour whitelister l'adresse IP
```consol
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: whitelist
spec:
  ipWhiteList:
    sourceRange:
      - <public IP adress>
```

Ajout d'une annotation sur ingress.yaml pour le middleware whitelist
``traefik.ingress.kubernetes.io/router.middlewares: default-auth@kubernetescrd,default-whitelist@kubernetescrd``

Quand Traefik est derrière un load balancer, il ne peut pas récupérer la vraie IP du client externe ayant inité la requête en vérifiant l'adresse IP distante, il voit l'adresse du load balancer à la place (reverse proxy). Il faut modifier cela pour permettre d'envoyer le traffic externe sur le noeud kubernetes où le traffic est entré.
``kubectl patch svc traefik -p '{"spec":{"externalTrafficPolicy":"Local"}}'``

## Chapitre 7 : Mettre en place un certificat TLS sur Traefik lié au domaine avec redirection en https

https://traefik.io/blog/secure-web-applications-with-traefik-proxy-cert-manager-and-lets-encrypt/

Installation de cert manager
``kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.1/cert-manager.yaml``

Création du fichier créant le certificat
```consol
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
 name: certmanager
spec:
 acme:
   email: <user.email>
   server: https://acme-v02.api.letsencrypt.org/directory
   privateKeySecretRef:
     name: cert-account-key
   solvers:
     - http01:
         ingress:
           class: traefik
```

Ajout d'une annotation sur ingress.yaml
``cert-manager.io/issuer: "certmanager"``

Ajout du TLS dans ingress.yaml
```consol
spec:
    tls:
        - hosts:
            - voteapp.simplon-celia.space
          secretName: tls-vote-ingress-http
```

https://aqibrahman.com/set-up-traefik-kubernetes-ingress-for-http-and-https-with-redirect-to-https

Redirection de http à https
```consol
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: redirect
spec:
  redirectScheme:
    scheme: https
    permanent: true
```

Ajout du middleware de redirection dans ingress.yaml
``traefik.ingress.kubernetes.io/router.middlewares: default-auth@kubernetescrd,default-whitelist@kubernetescrd,default-redirect@kubernetescrd``

## Chapitre 8 : Mettre en place une authentification avec certificats TLS client

Générer une autorité de certificat
```consol
openssl genrsa -des3 -out 'ca.key' 4096
openssl req -x509 -new -nodes -key 'ca.key' -sha384 -days 1825 -out 'ca.crt'
```

![](https://hackmd.io/_uploads/HyXfoozt3.png)

Ajout d'un secret avec le certificat ca.crt encodé en base 64
``cat ca.crt | base64 -w0``
```consol
apiVersion: v1
kind: Secret
metadata:
  name: client-auth-ca-cert
type: Opaque
data:
  ca.crt: "LS0tLS1CRUdJTi"
```

Ajout du certificat sur Traefik avec une TLS option qui vérifie le certificat client:
```option
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: client-cert
spec:
  minVersion: VersionTLS12
  maxVersion: VersionTLS13
  clientAuth:
    secretNames:
      - client-auth-ca-cert
    clientAuthType: RequireAndVerifyClientCert
  curvePreferences:
    - CurveP521
    - CurveP384
  cipherSuites:
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_AES_256_GCM_SHA384
    - TLS_CHACHA20_POLY1305_SHA256
  sniStrict: true
```

Ajout d'une annotation dans l'ingress 
``traefik.ingress.kubernetes.io/router.tls.options: default-client-cert@kubernetescrd``

Test sans certificat client avec `` curl -vk https://voteapp.simplon-celia.space/`` : erreur "TLS alert, bad certificate (554)"

![](https://hackmd.io/_uploads/SkyWf2Mtn.png)

Création de la clé et du certificat client
```consol
export DOMAIN=voteapp.simplon-celia.space
export ORGANIZATION=celia-simplon
export COUNTRY=fr

openssl genrsa -des3 -out "$DOMAIN.key" 4096
openssl req -new -key "$DOMAIN.key" -out "$DOMAIN.csr" -subj "/CN=$DOMAIN/O=$ORGANIZATION/C=$COUNTRY"

# Utilisation de ca.crt et ca.key du début
openssl x509 -sha384 -req -CA 'ca.crt' -CAkey 'ca.key' -CAcreateserial -days 365 -in "$DOMAIN.csr" -out "$DOMAIN.crt"
openssl pkcs12 -export -out "$DOMAIN.pfx" -inkey "$DOMAIN.key" -in "$DOMAIN.crt"
```

Test avec `` curl -vk https://voteapp.simplon-celia.space/ --cert "$DOMAIN.crt" --key "$DOMAIN.key"`` : erreur 401 unauthorized

Installer le certificat client sur Firefox
Paramètres > Vie Privée et Sécurité > Afficher les certificats > Vos certificats > Importer

Vérifier : ça marche

## Chapitre 9 : Mettre en place une authentification OAuth avec Google ID au lieu de l'authentification par mot de passe

![](https://hackmd.io/_uploads/HkjhVAMY3.png)

Création d'un identifiant client Oauth pour application web, avec URI de redirection autorisée sur https://voteapp.simplon-celia.space/_oauth et http://voteapp.simplon-celia.space/_oauth

Création d'un secret dans le cluster pour les identifiants google cloud
```consol
apiVersion: v1
kind: Secret
metadata:
  name: traefik-sso
type: Opaque
data:
  clientid: MTAwMjUxNDI4MTkwOC
  clientsecret: R09DU1BYLS03U2tuOHN
  secret: bmFiYW5
```

Création d'un déploiement et d'un service pour permettre l'identification sur le site en passant par l'authentification google à l'aide du secret précédent
```consol
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: traefik-sso
  name: traefik-sso
spec:
  progressDeadlineSeconds: 2147483647
  replicas: 1
  revisionHistoryLimit: 2147483647
  selector:
    matchLabels:
      app: traefik-sso
      name: traefik-sso
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: traefik-sso
        name: traefik-sso
    spec:
      containers:
      - env:
        - name: PROVIDERS_GOOGLE_CLIENT_ID
          valueFrom:
            secretKeyRef:
              key: clientid
              name: traefik-sso
        - name: PROVIDERS_GOOGLE_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              key: clientsecret
              name: traefik-sso
        - name: SECRET
          valueFrom:
            secretKeyRef:
              key: secret
              name: traefik-sso
        - name: COOKIE_DOMAIN
          value: simplon-celia.space
        - name: AUTH_HOST
          value: voteapp.simplon-celia.space
        - name: INSECURE_COOKIE
          value: "false"
        - name: WHITELIST
          value: celia.ouedrao@gmail.com
        - name: LOG_LEVEL
          value: debug
        image: thomseddon/traefik-forward-auth:2
        imagePullPolicy: Always
        name: traefik-sso
        ports:
        - containerPort: 4181
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-sso
spec:
  selector:
    app: traefik-sso
  ports:
  - protocol: TCP
    port: 4181
    targetPort: 4181
```

Création d'un middleware pour l'authentification
```consol
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: sso
spec:
  forwardAuth:
    address: http://traefik-sso:4181
    authResponseHeaders: 
        - "X-Forwarded-User"
    trustForwardHeader: true
```

Ajout du middleware dans les annotations de l'ingress
```consol
traefik.ingress.kubernetes.io/router.middlewares: default-whitelist@kubernetescrd,default-redirect@kubernetescrd,default-sso@kubernetescrd
```
