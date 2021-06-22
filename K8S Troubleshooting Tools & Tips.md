# Execute a command within a container from a Pod
```sh
kubectl exec <POD-NAME> -c <CONTAINER-NAME> -- <COMMAND>
kubectl exec --stdin --tty <POD-NAME> -c <CONTAINER-NAME> -- /bin/bash
```

# Delete mutiple pods using pattern
Delete all pods start with web-app
```
kubectl get pods -n default --no-headers=true | awk '/web-app/{print $1}'| xargs  kubectl delete -n default pod
```

# Busybox Curl Tool
Sometimes, you want to simulate a test from another Pod for troubleshooting purpose. This tool will deploy a pod with curl installed into the K8S cluster. 
After you exec into the pod, you can perform curl from the pod.
  
This command will deploy a interactive pod, once you exit from the pod, it will be removed automatically
```sh
kubectl run curl-<YOUR NAME> --image=radial/busyboxplus:curl -i --tty --rm
```  

This command will deploy a interactive pod, the pod will remain even if you exit the pod. Use the 2nd command to attach the pod back
```sh
kubectl run curl-<YOUR NAME> --image=radial/busyboxplus:curl -i --tty
kubectl attach <POD ID> -c curl-<YOUR NAME> -i -t
```


# Disable ValidatingWebhookConfiguration for Nginx Ingress controller
```sh
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

# Get a shell from a specific K8S node without SSH into the node (privilege container is required)
https://github.com/kvaps/kubectl-node-shell

# Create an ingress controller to an internal virtual network in Azure Kubernetes Service (AKS)
The ingress controller is configured on an internal, private virtual network and IP address. No external access is allowed

1. create a <b>internal-ingress.yaml</b> file
```sh
controller:
  service:
    loadBalancerIP: <choose-an-unused-IP-from-the-subnet>
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-internal: "true"
```

2. install the ingress controller using Helm
```sh
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the official stable repository
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress stable/nginx-ingress \
    --namespace ingress-basic \
    -f internal-ingress.yaml \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

3. verify the internal IP is assigned to the ingress
```sh
kubectl get service -l app=nginx-ingress --namespace ingress-basic
```

# Create a service and an ingress for ops4viya kibana
The kibana installed by ops4viya is accessible via NodePort by default. The following code creats a service and a ingress for kibana dashboard, so the service is accessible via the ingress.
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: sas-kibana
    role: kibana
  name: sas-kibana
  namespace: logging
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 5601
  selector:
    role: kibana
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  labels:
    app: sas-kibana
    role: kibana
  name: sas-kibana
  namespace: logging
spec:
  rules:
# update the host and path value accordingly
  - host: <ingress-URL>  
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          serviceName: sas-kibana
          servicePort: 80

```
Use http://<b>ingress-URL</b>/ to access the dashboard
