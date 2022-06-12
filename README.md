# basic_auth_ngnix_kubernetes
# How to add basic authentication to Kubernetes NGINX

If you dont want to get complex with using Okta/OUTAH, for a simple basic authentication you can use secrets and NGINX native in Kubernetes.

## Step 1: Create htpasswd file
Open a Linux terminal

    $ htpasswd -c auth "replace with your username, ie. test"
    New password: <bar>
    New password:
    Re-type new password:
    Adding password for user test
    
 ## Step 2: Convert htpasswd into a secret
 
    $ kubectl create secret generic basic-auth --from-file=auth
    secret "basic-auth" created
    
 ## Step 3: Examine the secret
 
    $ kubectl get secret basic-auth -o yaml
    apiVersion: v1
    data:
      auth: Zm9vOiRhcHIxJE9GRzNYeWJwJGNrTDBGSERBa29YWUlsSDkuY3lzVDAK
    kind: Secret
      metadata:
      name: basic-auth
      namespace: default
      type: Opaque
      
 ## Step 4: create a Ingress yaml file to use your basic-auth
 
 apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-with-basic-auth
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod" ==> If you are not using certs, this is no needed
    kubernetes.io/ingress.class: nginx  --> This is required if you have multiple ingress, check ingress.class in Kubernetes documentation
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - RedisInsights'
spec:
 tls:
  - hosts:
    - example.test.com  ==> replace with your domain
    secretName: ==> replace with your Cert secrect name
 rules:
  - host: example.test.com  ==> replace with your domain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service: 
            name: your-service
            port: 
              number: 80  --> your port
 

## Step 5: apply your yaml ingress and check you url address.

