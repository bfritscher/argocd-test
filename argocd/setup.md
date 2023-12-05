# setup
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


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