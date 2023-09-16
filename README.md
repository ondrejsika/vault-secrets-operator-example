# vault-secrets-operator-example

## Start Minikube

```
minikube start
```

## Install Vault

```
helm upgrade --install \
  vault \
  --repo https://helm.releases.hashicorp.com \
  vault \
  --namespace vault \
  --create-namespace \
  --values vault.values.yml
```

## Install Vault Secrets Operator

```
helm upgrade --install \
  vault-secrets-operator \
  --repo https://helm.releases.hashicorp.com \
  vault-secrets-operator \
  --namespace vault-secrets-operator-system \
  --create-namespace \
  --values vault-secrets-operator.values.yml
```

## Enable Kubernetes Auth

```
kubectl exec -ti vault-0 -n vault -- vault auth enable kubernetes
```

## Configure Kubernetes Auth

```
kubectl exec -ti vault-0 -n vault -- vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc.cluster.local:443"
```

## Create Secret Engine

```
kubectl exec -ti vault-0 -n vault -- vault secrets enable -path=test kv-v2
```

## Add Secret

```
kubectl exec -ti vault-0 -n vault -- vault kv put test/example username=foo password=bar
```

## Create Policy

```
kubectl exec -ti vault-0 -n vault -- vault policy write test-read - <<EOF
path "test/*" {
   capabilities = ["read"]
}
EOF
```

```
kubectl exec -ti vault-0 -n vault -- vault write auth/kubernetes/role/test-read \
   bound_service_account_names=default \
   bound_service_account_namespaces=default \
   policies=test-read \
   audience=vault \
   ttl=24h
```

## Create Vault Auth

```
kubectl apply -f vaultauth.yml
```

## Create Vault Secret

```
kubectl apply -f vaultstaticsecret.yml
```

## Check Secret

```
kubectl get secret example -o jsonpath='{.data.username}' | base64 --decode && echo
kubectl get secret example -o jsonpath='{.data.password}' | base64 --decode && echo
```
