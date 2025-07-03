# k8s-on-digital-ocean

Create a Kubernetes cluster [here](https://cloud.digitalocean.com/kubernetes/clusters?i=ebdc0a).

Add the cluster to your Kubeconfig
```
doctl kubernetes cluster list
doctl kubernetes cluster kubeconfig save 78287d0e-a0c4-4a9a-9dd5-a2cafc793ac7
```

Install ingress-nginx controller
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace --set controller.publishService.enabled=true
```

Create the CSI token secret in kube-system

Deploy the CSI controller and sidecar
```
kubectl apply -fhttps://raw.githubusercontent.com/digitalocean/csi-digitalocean/master/deploy/kubernetes/releases/csi-digitalocean-v4.9.0/{crds.yaml,driver.yaml,snapshot-controller.yaml}
```
Verify the creation of CSI that is being created in kube-system
```
kubectl get pods -n kube-system
```

Configure the actual PVC
```
kubectl apply -f manifests/test-storage.yaml
``` 

Verify Storage
```
kubectl get pvc
```

Add postgres chart (bitnami)
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Create PVCs - Create Persistent Volume 50GB for postgresDB
```
kubectl apply -f manifests/postgres-pv.yaml
```

Install postgres helm chart
```
helm install postgresdb bitnami/postgresql --set persistence.existingClaim=postgresql-pv-claim --set volumePermissions.enabled=true
```
Verify postgress POD is running
```
kubectl get pods
```

Get postgres password
```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresdb-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
echo POSTGRES_PASSWORD
```

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

Validate that the application works
```
kubectl get svc
kubectl port-forward svc/do-sample-app-service 8080:8080
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
- Install Cert Manager
- Install Monitoring solution
- Install Logging solution
- Improve security
- Set up CI/CD, Helm chart, etc
