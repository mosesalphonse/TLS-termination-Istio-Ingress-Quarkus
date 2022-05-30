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
