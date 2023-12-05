# setup
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-ssl-termination-at-ingress-controller
server.insecure: "true" in the argocd-cmd-params-cm ConfigMap 

kind: Ingress
metadata:
  name: argocd-server-http-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"

FIX gateway error if jwt payload too big
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/backend-protocol: HTTP
    nginx.ingress.kubernetes.io/client-body-buffer-size: 64k
    nginx.ingress.kubernetes.io/client-header-buffer-size: 100k
    nginx.ingress.kubernetes.io/http2-max-header-size: 96k
    nginx.ingress.kubernetes.io/large-client-header-buffers: 4 100k
    nginx.ingress.kubernetes.io/load-balance: ewma
    nginx.ingress.kubernetes.io/proxy-body-size: 150m
    nginx.ingress.kubernetes.io/proxy-buffer-size: 96k
    nginx.ingress.kubernetes.io/proxy-read-timeout: "1000"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "1000"
    nginx.ingress.kubernetes.io/server-snippet: | # this is where the magic happens
      client_header_buffer_size 100k;
      large_client_header_buffers 4 100k;

## access ui
kubectl port-forward svc/argocd-server -n argocd 8080:443

## keycloak 
https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/keycloak/

kubectl edit secret argocd-secret -n argocd
kubectl edit configmap argocd-cm -n argocd
kubectl edit configmap argocd-rbac-cm -n argocd

admin.enabled: "false"

  oidc.config: |
    name: Keycloak
    issuer: https://hec-iam.unil.ch/realms/hec
    clientID: argocd
    clientSecret: $oidc.keycloak.clientSecret
    insecureEnableGroups: true
    skipAudienceCheckWhenTokenHasNoAudience: true
    requestedScopes: ["openid", "profile", "email"]
  oidc.tls.insecure.skip.verify: "true"
  url: https://localhost:8080


  data:
  policy.csv: |
    g, kube-dev5-g, role:admin