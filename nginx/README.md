# Setting up a Kubernetes Nginx Ingress Controller locally

Here's a quick refresher on [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) is. According to official docs - 
> Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

Simply stated, *Ingress* is a [k8s object](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) that provides the ability to specific how to route/direct traffic to the [services](https://kubernetes.io/docs/concepts/services-networking/service)running inside the k8s cluster network (which is not accessible to the outside world).

In order for the Ingress objects to be useful, you will need to first setup an [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). k8s does not by default come with an Ingress Controller. There are multiple implementations of Ingress controllers [available](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

I have listed the steps required to set up the widely popular (ingress-nginx)[https://kubernetes.github.io/ingress-nginx/
], which is different from the [ingress controller](github.com/nginxinc/kubernetes-ingress) created by the Nginx folks. 

## Steps

### Preqs
You will need either [minikube](https://minikube.sigs.k8s.io/docs/start/) or [Docker For Mac](https://docs.docker.com/docker-for-mac/install/). I prefer *Docker for Mac* - it's has decent Dashboard and supports a single node k8s out of the box. 

1. See instructions to set up the 
Set up the (official) *Nginx Ingress Controller* for [Docker For Mac](https://kubernetes.github.io/ingress-nginx/deploy/#docker-for-mac) using the following command - 

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml`

2. Set up some services to test the ingress locally
All of the Pods and Service definitions for this example are available in the [api-services.yaml](api-services.yaml)

Run the following command to set up the objects defined in the resource manifest.

`kubectl apply -f api-services.yaml`

You can verify that the pods and the services are running using the following commands - 

`kubectl get pods`
```
NAME                                READY   STATUS      RESTARTS   AGE
api-1-echo-app                      1/1     Running     0          58m
api-2-echo-app                      1/1     Running     0          57m
```

`kubectl get services`
```
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
api-1-echo-service   ClusterIP   10.108.127.131   <none>        6500/TCP   57m
api-2-echo-service   ClusterIP   10.110.248.208   <none>        7500/TCP   57m
```

3. Set up the *Ingress* resources 
All of the Ingress definitions are in the [ingress.yaml](ingress.yaml).
As you can see from the ingress resource descriptions, I have set up two host based ingress rules
```
- host: api-1-service.svc.com
    http:
      paths:
        - path: /
          backend:
            serviceName: api-1-echo-service
            servicePort: 6500

  - host: api-2-service.svc.com
    http:
      paths:
        - path: /
          backend:
            serviceName: api-2-echo-service
            servicePort: 7500
```

The rules are pretty straight forward. In the above example, I have set up two rules, one for host `api-1-service.svc.com` and another one for `api-2-service.svc.com`. For each of the rules, specific *forwarding rules* are set up. As you can seem, all requests to `/` prefix is forwarded to the the appropriate service as specified by the `serviceName` and `servicePort` params in the spec. In the above example, all requests addressed to host *api-1-service.svc.com* (as indicated by the *spec.rules.host param) and path `/` (as indicated by the *spec.rules.http.paths.path param) is forwarded to the backend service *api-1-echo-service* listenin on port 6500.

Run the following command to set up the Ingress objects - 
`kubectl apply -f ingress.yaml`


4. Finally, set up the hostname in `/etc/hosts`
Add the following entries in the `/etc/hosts` file (you may need the appropriate permissions to modify this file)

`
127.0.0.1 api-1-service.svc.com
127.0.0.1 api-2-service.svc.com
`

5. If you point your browser to http://api-1-service.svc.com, you should see the following message `API One Invoked!`. Similarly, pointing your browser to http://api-2-service.svc.com, will display `API Two Invoked!`

6. That's it!








