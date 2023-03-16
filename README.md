# Knative Demo

This steps are based on a Kubernetes cluster built on Azure cloud

## Run a local development environment - https://knative.dev/docs/getting-started/quickstart-install/

OR

## Build a Basic Kubernetes cluster in Azure

```
az group create -l centralindia -n rg-knative-demo

az aks create -g rg-knative-demo -n aks-knative --load-balancer-sku basic --vm-set-type AvailabilitySet --node-vm-size Standard_B4ms --node-count 2
```

## Install the Knative Serving component:

Install the required components by running the command

```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.9.2/serving-crds.yaml

kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.9.2/serving-core.yaml
```

### To check the deployent - ```kubectl get pods -n knative-serving```



### Istio

```
kubectl apply -l knative.dev/crd-install=true -f https://github.com/knative/net-istio/releases/download/knative-v1.9.2/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.9.2/istio.yaml

kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.9.2/net-istio.yaml
```

### Fetch the External IP address or CNAME (Would be needed to configure DNS) by running the command:

```

kubectl --namespace istio-system get service istio-ingressgateway

```

## Configure DNS

Choose your Domain name, in my case I am using knative.te.nestdigital.com

So my DNS entry is like: *.knative.te.nestdigital.com => IP address found from the previous step


### Direct knative to use that domain

*** Replace knative.te.nestdigital.com with your domain suffix
```
kubectl patch configmap/config-domain --namespace knative-serving --type merge --patch '{"data":{"knative.te.nestdigital.com":""}}'
```
*** If you don't have a DNS server or DNS zone follow the Magic DNS section from this link - https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#configure-dns

## Configure to use TLS
### Install all cert-manager components

```

kubectl apply -f https://github.com/knative/net-certmanager/releases/download/knative-v1.9.2/release.yaml
```

### TLS with HTTP01

#### Install the net-http01 controller by running the command:


kubectl apply -f https://github.com/knative/net-http01/releases/download/knative-v1.9.2/release.yaml

#### Configure the certificate-class to use this certificate type by running the command:

```
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"certificate-class":"net-http01.certificate.networking.knative.dev"}}'
Enable autoTLS by running the command:


kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"auto-tls":"Enabled"}}'

```

### Create a public TLS certificate

#### Use - Let's Encrypt for free -  https://tecadmin.net/how-to-generate-lets-encrypt-ssl-using-certbot/

### Create a Kubernetes secret to hold your TLS certificate

```
kubectl create -n knative-serving secret tls default-cert  --key ./privkey1.pem --cert ./cert1.pem

kubectl patch configmap config-contour -n knative-serving \
  -p '{"data":{"default-tls-secret":"default-cert"}}'

```

## Install Knative Eventing

Install the required components by running the command

```
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.9.7/eventing-crds.yaml

kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.9.7/eventing-crds.yaml
```

### To check the deployent - ```kubectl get pods -n knative-eventing```


kn route describe hello

#### Create a new revision
```
kn service update hello --env TARGET=Knative
```

#### View Existing Revisions
```
kn revisions list
kubectl get revisions
```