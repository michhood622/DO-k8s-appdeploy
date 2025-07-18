# k8s-on-digital-ocean

- [x] Create a Kubernetes cluster [here](https://cloud.digitalocean.com/kubernetes/clusters?i=ebdc0a).
- [ ] Install Ingress Controller
- [ ] Create the CSI(Container Storage Interface)
- [ ] Configure Persistent Volumes
- [ ] Install Postgres
- [ ] Deploy Application
- [ ] Configure Ingress to point to Domain


Add the cluster to your Kubeconfig
```
doctl kubernetes cluster list
```
```
doctl kubernetes cluster kubeconfig save 78287d0e-a0c4-4a9a-9dd5-a2cafc793ac7
```
Check to see if cluster is up
```
kubectl get nodes
```

Install ingress-nginx controller
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
```
helm repo update
```
```
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace --set controller.publishService.enabled=true
```
Verify ingress has been created
```
kubectl get pods -n ingress-nginx
```

Create the CSI(Container Storage Interface) token secret in kube-system

First you need to goto Digital Ocean and create an API Token
[here](https://cloud.digitalocean.com/account/api/tokens)

Add the aforemetioned Token to the manifests/token-secret.yaml file and apply
```
kubectl apply -f manifests/token-secret.yaml
```

Deploy the CSI controller and sidecar
```
kubectl apply -fhttps://raw.githubusercontent.com/digitalocean/csi-digitalocean/master/deploy/kubernetes/releases/csi-digitalocean-v4.9.0/{crds.yaml,driver.yaml,snapshot-controller.yaml}
```
Verify the creation of CSI that is being created in kube-system
```
kubectl get pods -n kube-system
```
Check the Storage class that was created
```
kubectl get sc
```

Configure the actual PVC(Persistent Volume Claim) 5GB 
```
kubectl apply -f manifests/test-storage.yaml
``` 

Verify Storage
```
kubectl get pvc
```
Cerify the created storage in the DO webUI [here](https://cloud.digitalocean.com/volumes?i=5d1555)


Add postgres chart (bitnami) used to install Postgres
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
```
helm repo update
```

Create PVCs - Create Persistent Volume Claim 50GB for postgresDB
```
kubectl apply -f manifests/postgres-pv.yaml
```
Verify PVC Creation
```
kubectl get pvc
```

Install postgres helm chart
```
helm install postgresdb bitnami/postgresql --set persistence.existingClaim=postgresql-pv-claim --set volumePermissions.enabled=true
```
Verify postgress POD is running
```
kubectl get pods
```

helm should have created a secrets entry
```
kubectl get secrets
```

Get postgres password
```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresdb-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
echo $POSTGRES_PASSWORD
```

Inject the postgresDB password into the postgres-connection.yaml file


Create postgres connection string secret
```
Kubectl apply -f manifests/postgres-connection.yaml
```
Verify Secret was created

```
kubectl get secret
```

Deploy the application

```
Kubectl apply -f manifests/application.yaml
```
Verify POD Creation

```
kubectl get pods
```
```
kubectl logs do-sample-app-676767688b-8jrm7
```

Validate that the application works
```
kubectl get svc
```
```
kubectl port-forward svc/do-sample-app-service 8080:8080
```

Prove that we are levaraging our postgressDB
Open a new terminal window and restart our PODS

```
kubectl get deploy
```
```
kubectl rollout restart deploy/do-sample-app
```
```
kubectl get pods
```

Deploy the ingress
```
kubectl apply -f manifests/ingress.yaml
```

Point DNS CNAME to the ingress IP address
```
http://do-sample-app.michhood622.com/
```
Success :)


Next steps:
- Install Monitoring solution
- Install Logging solution
