# kubernetes-minikube

Minikube is a tool that lets you run Kubernetes locally. 
minikube runs a single-node Kubernetes cluster on your personal computer (including Windows, macOS and Linux PCs) so that you can try out Kubernetes, or for daily development work.

## Docker installation

### installation for Mac, Windows 10 Pro, Enterprise, or Education

https://www.docker.com/get-started

Choose Docker Desktop

### installation for Windows home

https://docs.docker.com/docker-for-windows/install-windows-home/

## Kuberntes Minikube installation

https://minikube.sigs.k8s.io/docs/start/

Minikube provides a dashboard (web portal). Access the dashboard using the following command:

```
minikube dashboard
```

## Download this project

This project contains a web service coded in Java, but the language doesn't matter. This project has already been built and the binary version is there:

First of all, download and uncompress the project: https://github.com/charroux/kubernetes-minikube

You can also use git: `git clone https://github.com/charroux/kubernetes-minikube`

Then move to the sud directory with `cd kubernetes-minikube/myservice` where a DockerFile is.

## Test this project using Docker

Build the docker image:
```
docker build -t myservice .
```

Check the image:
```
docker images
```

Start the container:
```
docker run -p 4000:8080 -t myservice
```

8080 is the port of the web service, while 4000 is the port for accessing the container. Test the web service using a web browser: http://localhost:4000 It displays hello.

Ctrl-C to stop the Web Service.

Check the containerID:
```
docker ps
```

Stop the container:
```
docker stop containerID
```

## Publish the image to the Docker Hub

Retreive the image ID:
```
docker images
```

Tag the docker image: 
```
docker tag imageID yourDockerHubName/imageName:version
```

Example: `docker tag 1dsd512s0d myDockerID/myservice:1`

Login to docker hub: 
```
docker login
```
or
```
docker login http://hub.docker.com
```
or 
```
docker login -u username -p password
```

Push the image to the docker hub:
```
docker push yourDockerHubName/imageName:version
```

Example: `docker push myDockerID/myservice:1`

## Create a kubernetes deployment from a Docker image

```
kubectl get nodes
```
**Result :** 
```
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   4m32s   v1.35.0
```

```
kubectl create deployment myservice --image=efrei/myservice:1
```

The image used comes from the Docker hub: https://hub.docker.com/r/efrei/myservice/tags

But you can use your own image instead.

Check the pod:
```
kubectl get pods
```
**Result :** 
```
NAME                         READY   STATUS              RESTARTS   AGE
myservice-5f87499856-db5nw   0/1     ContainerCreating   0          5s
```

Check if the state is running.

Get complete logs for a pods: 
```
kubectl describe pods
```
**Result :** 
```
kubectl describe pods
Name:             myservice-5f87499856-db5nw
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Wed, 25 Feb 2026 15:08:23 +0100
Labels:           app=myservice
                  pod-template-hash=5f87499856
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:           10.244.0.5
Controlled By:  ReplicaSet/myservice-5f87499856
Containers:
  myservice:
    Container ID:   docker://704262e7067ff83b61b71a76375eb4da3148d1e3fc2f9c29d1d8a730819f3f98
    Image:          claraaitm/myservice:1
    Image ID:       docker-pullable://claraaitm/myservice@sha256:a497fb25f69249647b722c5c3d2ac477c2412a766ba4accaa45b5914c7087a84
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 25 Feb 2026 15:08:49 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-t46rt (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-t46rt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
```

Retreive the IP address but notice that this IP address is ephemeral since a pods can be deleted and replaced by a new one.

Then retrieve the deployment in the minikube dashboard. 
Actually the Docker container is runnung inside a Kubernetes pods (look at the pod in the dashboard).
  
You can also enter inside the container in a interactive mode with:
```
kubectl exec -it podname -- /bin/bash
```

where podname is the name of the pods obtained with:
```
kubectl get pods
```

List the containt of the container with:
```
ls
```
**Result :** 
```
root@myservice-5f87499856-db5nw:/app# ls
app.jar
```

Don't forget to exit the container with:
```
exit
```

## Expose the Deployment through a service

A Kubernetes Service is an abstraction which defines a logical set of Pods running somewhere in the cluster, 
that all provide the same functionality. 
When created, each Service is assigned a unique IP address (also called clusterIP). 
This address is tied to the lifespan of the Service, and will not change while the Service is alive.

## Expose HTTP and HTTPS routes from outside the cluster to services within the cluster

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that’s outside of your cluster.

Kubernetes ServiceTypes allow you to specify what kind of Service you want. The default is ClusterIP.

Type values and their behaviors are:

* ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
* NodePort: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting NodeIP:NodePort.
* LoadBalancer: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
* ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record

## Expose HTTP and HTTPS route using NodePort

```
kubectl expose deployment myservice --type=NodePort --port=8080
```

Retrieve the service address:
```
minikube service myservice --url
```
**Result :** 
http://127.0.0.1:13874


This format of this address is `NodeIP:NodePort`.

Test this address inside your browser. It should display hello again.

Look from the NodeIP and the NodePort in the minikube dashboard.

**Results :** 
NodeIP : 192.168.49.2
NodePort : 31970 

## Scaling and load balancing

Check if the myservice deployment is running:

```
kubectl get deployments
```
**Result :** 
```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
myservice   1/1     1            1           31m
```

How many instance are actually running:

```
kubectl get pods
```

**Result :** 
```
NAME                         READY   STATUS    RESTARTS   AGE
myservice-5f87499856-db5nw   1/1     Running   0          31m
```

Start a second instance:

```
kubectl scale --replicas=2 deployment/myservice
```
```
kubectl get deployments
```
**Result :** 
```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
myservice   2/2     2            2           32m
```

and 

```
kubectl get pods
```
**Result :** 
```
NAME                         READY   STATUS    RESTARTS   AGE
myservice-5f87499856-db5nw   1/1     Running   0          32m
myservice-5f87499856-ps6qd   1/1     Running   0          25s
```

again

## Creating a Service of type LoadBalancer

Check if the myservice deployment is running:

```
kubectl get deployments
```

If a service is running in front of the deployment you must delete this service first in ordre to create a new one of kind LoadBalancer. So retreive the service using:

```
kubectl get services
```
And delete it:
```
kubectl delete service serviceName
```
```
kubectl expose deployment myservice --type=LoadBalancer --port=8080
```
```
minikube service myservice --url
```
Results : 
http://127.0.0.1:17844

Test in your web browser

## Rolling updates

Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.

To update the image of the application to version 2, use the set image subcommand, followed by the deployment name and the new image version:
```
kubectl set image deployments/my-deployment my-deployment=dockerHudId/my-image:v2
```
**Results : ** deployment.apps/myservice image updated

You can also confirm the update by running the rollout status subcommand:
```
kubectl rollout status deployments/my-deployment
```
**Result :** deployment "myservice" successfully rolled out

To roll back the deployment to your last working version, use the rollout undo subcommand:
```
kubectl rollout undo deployments/my-deployment
```
**Result :** deployment.apps/myservice rolled back

## Create a deployment and a service using a yaml file

Yaml files can be used instead of using the command `kubectl create deployment` and `kubectl expose deployment`

The yaml file for the deployment: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-deployment.yml

The yaml file for the node port service: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-service.yml

The yaml file for the node port service: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-loadbalancing-service.yml

Apply the deployment:
```
kubectl apply -f myservice-deployment.yml
```
**Result :** deployment.apps/myservice configured

Apply the node port service: 
```
kubectl apply -f myservice-service.yml
```
**Result :** service/myservice configured

or 

Apply the service of type loadbalancer:
```
kubectl apply -f myservice-loadbalancing-service.yml
```
Then test if it works as expected.
**Result :** kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
myservice-746766f779-kmmsc   1/1     Running   0          2m38s

# Routing rule to a service using Ingress

You can use Ingress to expose your Service. 
Ingress is not a Service type, but it acts as the entry point for your cluster. 
It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. 
An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.

## Set up Ingress on Minikube with the NGINX Ingress Controller

Enable the NGINX Ingress controller: 

```
minikube addons enable ingress
```
**Result :**
💡  ingress est un addon maintenu par Kubernetes. Pour toute question, contactez minikube sur GitHub.
Vous pouvez consulter la liste des mainteneurs de minikube sur : https://github.com/kubernetes/minikube/blob/master/OWNERS
💡  Après que le module est activé, veuiller exécuter "minikube tunnel" et vos ressources ingress seront disponibles à "127.0.0.1"
    ▪ Utilisation de l'image registry.k8s.io/ingress-nginx/controller:v1.14.1
    ▪ Utilisation de l'image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.6.5
    ▪ Utilisation de l'image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.6.5
🔎  Vérification du module ingress...
🌟  Le module 'ingress' est activé

Verify that the NGINX Ingress controller is running:
```
kubectl get pods -n ingress-nginx
```
**Result :** 
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-wk5kc        0/1     Completed   0          106s
ingress-nginx-admission-patch-7fjc4         0/1     Completed   1          106s
ingress-nginx-controller-8675c6b56f-x5wfw   1/1     Running     0          106s

Create a Deployment and expose it as a NodePort (not a loadbalancer).

Check if it works.

A yaml file for ingress: https://github.com/charroux/kubernetes-minikube/blob/main/ingress.yml

```
kubectl apply -f ingress.yml
```
**Result : ** ingress.networking.k8s.io/example-ingress created

Retrieve the IP address of Ingress: 

```
kubectl get ingress
```

```
NAME                 CLASS    HOSTS                  ADDRESS        PORTS   AGE

example-ingress      nginx   myservice.info         192.168.64.2   80      18m
```

**Result :**
NAME              CLASS   HOSTS            ADDRESS   PORTS   AGE
example-ingress   nginx   myservice.info             80      22s


On Linux: edit the `/etc/hosts` file and add at the bottom values for: 

IngressAddress myservice.info

Where address is given by:
```
minikube ip
```

On Mac: edit the `/etc/hosts` file and add at the bottom values for: 

127.0.0.1 myservice.info


Then check in your Web browser: 

http://myservice.info/

On Windows : edit the `c:\windows\system32\drivers\etc\hosts` file, add 

`127.0.0.1 myservice.info`	

Enable a tunnel for Minikube:

```
minikube addons enable ingress-dns
```
** Result :** 
💡  ingress-dns est un addon maintenu par minikube. Pour toute question, contactez minikube sur GitHub.
Vous pouvez consulter la liste des mainteneurs de minikube sur : https://github.com/kubernetes/minikube/blob/master/OWNERS
💡  Après que le module est activé, veuiller exécuter "minikube tunnel" et vos ressources ingress seront disponibles à "127.0.0.1"
    ▪ Utilisation de l'image docker.io/kicbase/minikube-ingress-dns:0.0.4
🌟  Le module 'ingress-dns' est activé


```
minikube tunnel
```
**Result : **
✅  Tunnel démarré avec succès

📌  REMARQUE : veuillez ne pas fermer ce terminal car ce processus doit rester actif pour que le tunnel soit accessible...

❗  Accéder aux ports inférieurs à 1024 peut échouer sur Windows avec les clients OpenSSH antérieurs à v8.1. Pour plus d'information, voir: https://minikube.sigs.k8s.io/docs/handbook/accessing/#access-to-ports-1024-on-windows-requires-root-permission
🔗  Tunnel de démarrage pour le service example-ingress.

Then check in your Web browser: 

http://myservice.info/


Create a second deployment and its service, then add a new route to the ingress.yml file.

## Delete resources

```
kubectl delete services myservice
```
```
kubectl delete deployment myservice
```

