# aps-chart

Alfresco Process Services (APS) v1.9 Kubernetes Helm Chart with Postgresql Database 

Pre-requisites:

* Kubernetes cluster (minikube should be OK)
* Helm 
* Nginx


Instructions:

1. Create activiti.lic Kubernetes secret from activiti.lic file. Run something like `kubectl create secret generic activiti.lic --from-file=./activiti.lic` in the namespace where you want to install APS.

2. Find out your Kubernetes cluster hostname or public IP address. For minikube run `minikube ip` command.

3. Modify `activiti-app.ingress.hosts` entry value with `nano ./aps/values.yaml`. Use aps.W.X.Y.Z.nip.io wildcard DNS as a host entry if you don't have public domain name for your cluster. You may also need to change ingress annotations for your Kubernetes cluster nginx controller to create proper Ingress resource.

4. Run `make -C aps install`

5. Check deployment status `make -C aps status`

6. Open browser and go to `http://aps.W.X.Y.Z.nip.io/activiti-app`

7. Default Login/Password: `admin@app.activiti.com`/`admin`

Notes:

The Aps chart template will auto-generate postgresql password in `aps-postgresql` Kubernetes secret that will be injected at runtime into deployments. 

You may specify your own Postgresql password using `postgresPassword` value for aps-postgresql secret using helm install command line options.

License:

Apache Software License 2.0