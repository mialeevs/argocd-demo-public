# Install and Configure ArgoCD

> A kubernetes cluster to install and configure ArgoCD


> Install ArgoCD
```bash
# Non HA installation
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.13.2/manifests/install.yaml

# HA installation
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.13.2/manifests/ha/install.yaml
```

> Get the password - Method-1
```bash
# Get all the secrets in argocd namespace
kubectl get secrets -n argocd

# Get the password from the secret file
kubectl get secret -n argocd argocd-initial-admin-secret -o yaml > initial_admin.yaml

cat initial_admin.yaml

apiVersion: v1
data:
  password: NU9KUHdFOUVFcmt5eFJtQQ==
kind: Secret
metadata:
  creationTimestamp: "2023-07-11T00:58:04Z"
  name: argocd-initial-admin-secret
  namespace: argocd
  resourceVersion: "54536"
  uid: de62ff7c-d47a-45f6-92f9-b9a5181e51ed
type: Opaque

# Copy the password and decode it
echo -n 'NU9KUHdFOUVFcmt5eFJtQQ==' | base64 --decode

5OJPwE9EErkyxRmA
```

> Get the password - Method-2
```bash
# Get the password from the secret file
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```


> Expose the service using nodePort
```bash
# List all the services in the argocd namespace
kubectl get svc -n argocd

# Edit the service and change the service type from ClusterIP to NodePort
kubectl edit -n svc argocd argocd-server

# Edit at the bottom of the file
ports:
  - name: http
    nodePort: 32399
    port: 80
    protocol: TCP
    targetPort: 8080
  - name: https
    nodePort: 32400
    port: 443
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: NodePort


# Or direclty edit using the patch command
kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"NodePort"}}'
kubectl patch svc argocd-server -n argocd --type='json' -p='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value": 32399}]'
kubectl patch svc argocd-server -n argocd --type='json' -p='[{"op": "replace", "path": "/spec/ports/1/nodePort", "value": 32400}]'

```
> Login to the UI

```bash
# Login to the UI using url

http://<workernode_ip>:32400

```

> Download the ArgoCD CLI
```bash
# ArgoCD cli resources
https://github.com/argoproj/argo-cd/releases/latest

# For linux based OS I use below link to download
wget https://github.com/argoproj/argo-cd/releases/download/v2.13.2/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# Ue below link to download onto Win 11
https://github.com/argoproj/argo-cd/releases/download/v2.13.2/argocd-windows-amd64.exe

```


