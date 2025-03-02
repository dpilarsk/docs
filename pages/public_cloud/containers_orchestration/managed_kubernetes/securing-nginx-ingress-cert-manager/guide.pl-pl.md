---
title: Secure a Nginx Ingress with cert-manager on OVHcloud Managed Kubernetes
excerpt: 'Find out how to secure a Nginx Ingress with cert-manager on OVHcloud Managed Kubernetes'
updated: 2024-04-11
---

## Objective

In this tutorial you are going to:

- deploy an application (a `Deployment` and a ClusterIP `Service`) on your OVHcloud Managed Kubernetes cluster
- install the Nginx Ingress Controller
- deploy an Ingress to configure the Nginx Ingress Controller to use SSL/TLS through cert-manager

## Before you begin

This tutorial presupposes that you already have a working OVHcloud Managed Kubernetes cluster, and some basic knowledge of how to operate it. If you want to know more on those topics, please look at the [OVHcloud Managed Kubernetes Service Quickstart guide](/pages/public_cloud/containers_orchestration/managed_kubernetes/deploying-hello-world).

You need to have [Helm](https://docs.helm.sh/){.external} installed on your workstation and your cluster. Please refer to our tutorial on [How to install Helm on OVHcloud Managed Kubernetes Service](/pages/public_cloud/containers_orchestration/managed_kubernetes/installing-helm).

You also need to install [cert-manager](/pages/public_cloud/containers_orchestration/managed_kubernetes/installing-cert-manager) on your OVHcloud Managed Kubernetes.

## Instructions

### Deploying the application

In this guide you will deploy an application that runs a HTTP server and displays a web page.

First, create a `deployment.yaml` file with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment
  labels:
    app: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: ovhplatform/hello
        ports:
        - containerPort: 80
```

This YAML deployment manifest file defines that our application, based on the `ovhplatform/hello:latest` image will be deployed with 1 replica (1 pod).

Then, create a `svc.yaml` file with the following content to define our service (a service exposes a deployment):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world
  labels:
    app: hello-world
spec:
  ports:
  - port: 80
    name: http
  selector:
    app: hello-world
```

Apply the deployment and service manifest files to your cluster with the following commands:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f svc.yaml
```

Output should be like this:

```console
$ kubectl apply -f deployment.yaml
deployment.apps/hello-world-deployment created

$ kubectl apply -f svc.yaml
service/hello-world created
```

You can verify if your application is running and service is created by running the following commands:

```bash
kubectl get pod -l app=hello-world
kubectl get svc -l app=hello-world
```

Output should be like this:

```console
$ kubectl get pod -l app=hello-world
NAME                                      READY   STATUS    RESTARTS   AGE
hello-world-deployment-559d658ffb-qtxnl   1/1     Running   0          61s

$ kubectl get svc -l app=hello-world
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
hello-world   ClusterIP   10.3.213.111   <none>        80/TCP    68s
```

### Installing the Nginx Ingress Controller Helm chart

For this tutorial, we are using the [Nginx Ingress Controller Helm chart](https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx){.external} found on its own Helm repository.

The chart is fully configurable, but here we are using the default configuration.

Add the Ingress Nginx Helm repository:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

These commands will add the Ingress Nginx Helm repository to your local Helm chart repository and update the installed chart repositories:

```console
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update
"ingress-nginx" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "nvidia" chart repository
...
...Successfully got an update from the "ingress-nginx" chart repository
...
Update Complete. ⎈Happy Helming!⎈
```

Install the latest version of Ingress Nginx with `helm install` command:

```bash
helm -n ingress-nginx install ingress-nginx ingress-nginx/ingress-nginx --create-namespace
```

The install process will begin and a new `ingress-nginx` namespace will be created.

```console
$ helm -n ingress-nginx install ingress-nginx ingress-nginx/ingress-nginx --create-namespace

NAME: ingress-nginx
LAST DEPLOYED: Thu Apr 11 10:54:34 2024
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the load balancer IP to be available.
You can watch the status by running 'kubectl get service --namespace ingress-nginx ingress-nginx-controller --output wide --watch'

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

As the `LoadBalancer` creation is asynchronous, and the provisioning of the load balancer can take several minutes, you will surely get a `<pending>` `EXTERNAL-IP`. 

If you try again in a few minutes you should get an `EXTERNAL-IP`:

```console
$ kubectl get svc -n ingress-nginx ingress-nginx-controller
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.3.232.157   xx.xx.xx.xx   80:30903/TCP,443:31546/TCP   19h
```

You can then access your `nginx-ingress` at `http://[YOUR_LOAD_BALANCER_IP]` via HTTP or `https://[YOUR_LOAD_BALANCER_IP]` via HTTPS.

### Configuring the Nginx Ingress Controller to use SSL/TLS

After installing [cert-manager](/pages/public_cloud/containers_orchestration/managed_kubernetes/installing-cert-manager) on your OVHcloud Manager Kubernetes cluster, if you followed the guide, you should have two running ClusterIssuer, one for production and one for staging/dev usages:

```console
$ kubectl get clusterissuer
NAME                  READY   AGE
letsencrypt-prod      True    7s
letsencrypt-staging   True    7s
```

If it's not the case, create a `issuer.yaml` file with the following content:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: [YOUR_EMAIL]
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: [YOUR_EMAIL]
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
```

> [!primary]
> 
> `nginx` is the name of the Ingress resource you are going to create.

And deploy it:

```console
kubectl apply -f issuer.yaml
```

To certificate a resource, the Ingress in our case, we will use annotations.
Thanks to that, Cert-manager will create the `Certificate` resource that represents a human readable definition of a certificate request. Cert-manager uses this input to generate a private key and `CertificateRequest` resource in order to obtain a signed certificate from an `Issuer` or `ClusterIssuer`. The signed certificate and private key are then stored in the specified `Secret` resource.

At this step, you need to deploy an Ingress resource and configure it to use the SSL/TLS terminaison.

Create an `ingress-tls.yaml` file with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
  name: ingress
  namespace: default
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - [YOUR_DN]
      secretName: hello-world-tls # < cert-manager will store the created certificate in this secret.
  rules:
  - host: [YOUR_DN]
    http:
      paths:
      - backend:
          service:
            name: hello-world
            port:
              number: 80
        path: /
        pathType: Prefix
```

> [!primary]
>
> Don't forget to replace `[YOUR_DN]` with your domain name.

In this manifest file you can see that we define a Nginx Ingress resource with several annotations.
For more information about the annotations, please refer to the [Securing Ingress Resources in cert-manager documentation](https://cert-manager.io/docs/usage/ingress/#supported-annotations).

Apply the ingress manifest files to your cluster with the following commands:

```bash
kubectl apply -f ingress-tls.yaml
```

Output should be like this:

```console
$ kubectl apply -f ingress-tls.yaml
ingress.networking.k8s.io/ingress configured
```

At this step, a `Certificate` resource has been created:

```console
$ kubectl get certificate
NAME              READY   SECRET            AGE
hello-world-tls   False   hello-world-tls   111s
```

You can display the events of the certificate to check if the certificate has been correctly created and if it creates the necessary `CertificateRequest`:

```console
$ kubectl describe certificate
Name:         hello-world-tls
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  cert-manager.io/v1
Kind:         Certificate
Metadata:
  Creation Timestamp:  2022-06-27T11:14:40Z
  Generation:          1
  ...
Events:
  Type    Reason     Age    From                                       Message
  ----    ------     ----   ----                                       -------
  Normal  Issuing    8m57s  cert-manager-certificates-trigger          Issuing certificate as Secret does not exist
  Normal  Generated  8m57s  cert-manager-certificates-key-manager      Stored new private key in temporary Secret resource "hello-world-tls-krzvj"
  Normal  Requested  8m57s  cert-manager-certificates-request-manager  Created new CertificateRequest resource "hello-world-tls-4fzl2"
```

And you can check that a `CertificateRequest`, an `Order` and a `Challenge` resource have been created:

```console
$ kubectl describe certificaterequest
Name:         hello-world-tls-4fzl2
Namespace:    default
Labels:       <none>
Annotations:  cert-manager.io/certificate-name: hello-world-tls
              cert-manager.io/certificate-revision: 1
              cert-manager.io/private-key-secret-name: hello-world-tls-krzvj
API Version:  cert-manager.io/v1
Kind:         CertificateRequest
Metadata:
  Creation Timestamp:  2024-01-18T14:34:41Z
  Generate Name:       hello-world-tls-
  Generation:          1
  ...
Events:
  Type    Reason           Age   From                                          Message
  ----    ------           ----  ----                                          -------
  Normal  cert-manager.io  18m   cert-manager-certificaterequests-approver     Certificate request has been approved by cert-manager.io
  Normal  OrderCreated     18m   cert-manager-certificaterequests-issuer-acme  Created Order resource default/hello-world-tls-4fzl2-2471308949
  Normal  OrderPending     18m   cert-manager-certificaterequests-issuer-acme  Waiting on certificate issuance from order default/hello-world-tls-4fzl2-2471308949: ""
```

```console
$ kubectl describe order
Name:         hello-world-tls-4fzl2-2471308949
Namespace:    default
Labels:       <none>
Annotations:  cert-manager.io/certificate-name: hello-world-tls
              cert-manager.io/certificate-revision: 1
              cert-manager.io/private-key-secret-name: hello-world-tls-krzvj
API Version:  acme.cert-manager.io/v1
Kind:         Order
Metadata:
  Creation Timestamp:  2024-01-18T14:34:41Z
  Generation:          1
  ...
Events:
  Type    Reason   Age   From                 Message
  ----    ------   ----  ----                 -------
  Normal  Created  16m   cert-manager-orders  Created Challenge resource "hello-world-tls-4fzl2-2471308949-3172376200" for domain "example.com"
```

```console
$ kubectl describe challenge
Name:         hello-world-tls-4fzl2-2471308949-3172376200
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  acme.cert-manager.io/v1
Kind:         Challenge
Metadata:
  Creation Timestamp:  2024-01-18T14:34:42Z
  Finalizers:
    finalizer.acme.cert-manager.io
  Generation:  1
  ...
Events:
  Type    Reason     Age   From                     Message
  ----    ------     ----  ----                     -------
  Normal  Started    16m   cert-manager-challenges  Challenge scheduled for processing
  Normal  Presented  16m   cert-manager-challenges  Presented challenge using HTTP-01 challenge mechanism
```

You now need to map the Domain Name (DN) and the Load Balancer. In order to do this, create an `A-record` for `[YOUR_DN]` (your domain name ;-) mapped to the value of `$INGRESS_URL`.

Wait until the challenge is resolved:

```
dig +short [YOUR_DN]
```

Describe the certificate again and wait until you see "Certificate issued successfully" when you describe the certificate.

### Certificate renewal

Cert-manager is a very powerful tool that manages certificate creation and also automatic certificate renewal.

Once an X.509 certificate has been issued, cert-manager will calculate the renewal time for the `Certificate` and will set the Certificate's `RenewalTime` status to the time when the renewal is attempted.

You can check your certificate renewal:

```console
$ kubectl get certificate -o yaml | grep renew
    renewalTime: "2024-03-18T14:05:35Z"
```

## Go further

- If you need training or technical assistance to implement our solutions, contact your sales representative or click on [this link](https://www.ovhcloud.com/pl/professional-services/) to get a quote and ask our Professional Services experts for assisting you on your specific use case of your project.

- Join our community of users on <https://community.ovh.com/en/>.
