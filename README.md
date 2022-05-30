# TLS-termination-Istio-Ingress- Gateway - Quarkus
This Repo may help you to set up TLS termination on Istio Ingress Gateway with Self Signed and CA signed for Quarkus workloads

## POC:
<li>
Quarkus Hello World workload with Istio Ingress Gateway
</li>
<li>
TLS termination in Istio Ingress Gateway with Self Singed certificate (for Non-prod)
</li>
<li>
TLS termination in Istio Ingress Gateway with CA signed certificate (for prod)
</li>

## Prerequisite

<li>
Kubernetes Cluster (tested on gke)
 </li>
 <li>
Istio Instalation (tested on istio-1.13.4)
</li>
<li>
Registerd Domain
</li>
<li>
CA Signed certificate on the above domain
</li>

## Deploy Workload

```
git clone https://github.com/mosesalphonse/TLS-termination-Istio-Ingress-Quarkus.git

cd TLS-termination-Istio-Ingress-Quarkus

cd Workload

mvn clean package -Dmaven.test.skip=true -Dquarkus.container-image.push=true

```
Note : Verify the image in the GCR


```
cd ..

kubectl apply -f workload/yamls/quarkus-deployment.yaml

kubectl apply -f workload/yamls/gateway.yaml

kubectl apply -f workload/yamls/VS.yaml

```
### Verify (There is no TLS applied for Istio Ingress Gateway)

```
export INGRESS_IP=$(kubectl get svc istio-ingressgateway -n istio-system -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

curl $INGRESS_IP/hello

curl $INGRESS_IP/hello/greeting/{name}

curl $INGRESS_IP/hello/greeting/{count}/{name}

curl $INGRESS_IP/hello/stream/{count}/{name}


```
## Apply Self Signed certificate (usefull for Non-production env)

### Create Self Signed Certificate

```
export DOMAIN_NAME={domainname}

openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=$DOMAIN_NAME Inc./CN=$DOMAIN_NAME' -keyout $DOMAIN_NAME.key -out $DOMAIN_NAME.crt

openssl req -out sash.$DOMAIN_NAME.csr -newkey rsa:2048 -nodes -keyout sash.$DOMAIN_NAME.key -subj "/CN=$DOMAIN_NAME/O=hello from $DOMAIN_NAME"

openssl x509 -req -days 365 -CA $DOMAIN_NAME.crt -CAkey $DOMAIN_NAME.key -set_serial 0 -in sash.$DOMAIN_NAME.csr -out sash.$DOMAIN_NAME.crt

```
