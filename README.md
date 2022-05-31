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

curl $INGRESS_IP/hello/greeting/Sashvin

curl $INGRESS_IP/hello/greeting/10/Moses

curl $INGRESS_IP/hello/stream/15/Sash


```
## Apply Self Signed certificate (usefull for Non-production env)

### Create Self Signed Certificate

```
mkdir cert-self-signed

cd cert-self-signed

export DOMAIN_NAME={domainname, example sashvinmoses.tk}

openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=$DOMAIN_NAME Inc./CN=$DOMAIN_NAME' -keyout $DOMAIN_NAME.key -out $DOMAIN_NAME.crt

openssl req -out sash.$DOMAIN_NAME.csr -newkey rsa:2048 -nodes -keyout sash.$DOMAIN_NAME.key -subj "/CN=$DOMAIN_NAME/O=hello from $DOMAIN_NAME"

openssl x509 -req -days 365 -CA $DOMAIN_NAME.crt -CAkey $DOMAIN_NAME.key -set_serial 0 -in sash.$DOMAIN_NAME.csr -out sash.$DOMAIN_NAME.crt

```
### Create secrets in Kubernetes

```
kubectl create -n istio-system secret tls sash-credential --key=sash.sashvinmoses.tk.key --cert=sash.sashvinmoses.tk.crt

```
### Update Port, Protocol, secrets and Host in Gateway

```
cd ..

kubectl apply -f workload/yamls/gateway-self-signed.yaml

kubectl apply -f workload/yamls/VS-TLS.yaml

```

### Veify (with Self Signed certificate)

```
cd cert-self-signed

export INGRESS_IP=$(kubectl get svc istio-ingressgateway -n istio-system -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

curl -H "Host:sashvinmoses.tk" --resolve "sashvinmoses.tk:443:$INGRESS_IP" --cacert sash.sashvinmoses.tk.crt "https://sashvinmoses.tk:443/hello" -v

curl -H "Host:sashvinmoses.tk" --resolve "sashvinmoses.tk:443:$INGRESS_IP" --cacert sash.sashvinmoses.tk.crt "https://sashvinmoses.tk:443/hello/greeting/{name}" -v

curl -H "Host:sashvinmoses.tk" --resolve "sashvinmoses.tk:443:$INGRESS_IP" --cacert sash.sashvinmoses.tk.crt "https://sashvinmoses.tk:443/hello/greeting/{count}/{name}" -v

curl -H "Host:sashvinmoses.tk" --resolve "sashvinmoses.tk:443:$INGRESS_IP" --cacert sash.sashvinmoses.tk.crt "https://sashvinmoses.tk:443/hello/stream/{count}/{name}" -v

```
## Apply CA Signed certificate (For Production Systems)

### Some hint to create your own free domain


Sign in into https://my.freenom.com/ using your google account

Register a new domain, refer the below screenshot

![image](https://user-images.githubusercontent.com/16347988/171029594-a86d9b9c-b3ae-4617-bd39-fb5273c33c8a.png)

Makesure you checkout so that you will get free domain. 


### Some hint to create free TLS certificate

a) Signin into https://zerossl.com/

b) Click New Certificate as shown in the below screenshot below
![image](https://user-images.githubusercontent.com/16347988/171122979-0243aff9-05d2-4c08-8952-24b08db05360.png)

c) Follow the steps, you may choose different verification methods, If you want to use DNS(CNAME), please refer the below steps

![image](https://user-images.githubusercontent.com/16347988/171123849-995f09f9-b2d6-4295-bd95-706d86e35ce9.png)
  
   i) Copy the 'Name' and 'Point To' values, these values to be configured in the DNS provider, for this example, these values to be updated in 'https://my.freenom.com/' as shown in the below screenshot
   
   ![image](https://user-images.githubusercontent.com/16347988/171124870-e844b51d-af40-4cc2-a9da-76585eef4040.png)

Note: CNAME record values shoule be copied from the TLS provider as I shown in the above screenshot. A record value is your Istio Ingress Gateway's external IP address.

d) Once you have updated CNAME on your DNS provider, it may take around 15 minitues to reflect the changes, then you can able to verify it. Once you have completed all the steps, you can download the certificate bundles which consist of Certificate, Private key and CA bundle.


Note: Make sure you should have CA(Certificate Authority) signed certificate and private key before starting this step. The above hint may guide you to obtain new domain and TLS certificate.


### Create secrets in Kubernetes

```
cd ..

mkdir cert-ca

cd cert-ca

```
Note: Make sure upload your certificate, private key in this folder. You may use secured vault to keep these sensitive values as well like hashicorp, AWS secrets manager etc

```
kubectl create -n istio-system secret tls sashvin-credential --key=private.key --cert=certificate.crt

```
### Update Port, Protocol, secrets and Host in Gateway

```
cd ..

kubectl apply -f workload/yamls/gateway-ca.yaml

```
### Veify (with CA Signed certificate) in Web Browser


```

https://sashvinmoses.tk/hello
https://sashvinmoses.tk/hello/greeting/Moses
https://sashvinmoses.tk/hello/greeting/10/sashvin
https://sashvinmoses.tk/hello/stream/15/Moses

```
