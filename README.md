# Deployment repo

Deployment of the container using Kustomize.

> **DOCS**: Read all the steps from the Deployments in K8S
> * https://gitlab.com/supercash/platform/k8s/k8s-eks-setup/-/wikis/3.-Deployments-in-K8S/5.-Deploy-a-Kubernetes-Environment

We are supposed to have the following environmnets:

| Env | Branch  | When updated |
| --  | --      | -- |
| `dev` | `develop` | When pushed to the repository |
| `qal` | `feature/**` | When creating branch for testing, bugfixes |
| `stg` | `PR(develop->master)` | When a new PR is built against master |
| `prd` | `master` | When pushed to the repository (or merged from PR) |

## Steps to setup

> **Details**: https://gitlab.com/supercash/platform/k8s/k8s-eks-setup/-/wikis/3.-Deployments-in-K8S/1.-Create-Gitlab-Container-Secret

* Create a Base
* Create an Env
* Deploy the ArgoCD ssh secret
* Create the env ArgoCD App

# Base

Implementation of all the shared properties of a deployment that is shared among all environments.

> **REF**: https://gitlab.com/supercash/platform/k8s/k8s-eks-setup/-/wikis/3.-Deployments-in-K8S/3.-Create-Base-Deployment

# Kustomize - Base

* Secrets to pull docker images
* Secrets of environmnet variables (shared)
* Base Deployment, Service

## Testing Kustonize Base

Always run the command to verify the k8s objects generated.

* Use `kubectl kustomize base`

## Testing Kustomize Env

* Use `kubectl kustomize env/xyz`

## Apply

* Use `kubectl apply -k env/xyz`

## Delete 

* Use `kubectl delete -k env/xyz`

# HPA Setup

> **REF**: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

* Only included in the prd segument:
  * stg: NOT ADDED YET
  * prd: so far installed

## Load Generator

> **NOTE**: You need to run 3 terminals: k9s logs, load generator, hpa status.

* Just run the following in one terminal:

### Generator pod

* Retrieve the env var that points to the service on port 80.

```console
$ kubectl -n supercash-aws-sae1-prdt-prd-prd run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "env"
SUPERCASH_AUTH_SERVICE_AWS_SAE1_PRDT_PRD_PRD_PORT_80_TCP=tcp://10.100.47.73:80
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.100.0.1:443
SUPERCASH_POSTGRES_SERVER_AWS_SAE1_PRDT_PRD_PRD_SERVICE_PORT_POSTGRESQL=5432
SUPERCASH_PAYMENT_SERVICE_AWS_SAE1_PRDT_PRD_PRD_PORT_80_TCP=tcp://10.100.74.125:80
HOSTNAME=load-generator
SUPERCASH_PARKINGLOT_SERVICE_AWS_SAE1_PRDT_PRD_PRD_SERVICE_HOST=10.100.69.238
SUPERCASH_GATEWAY_SERVICE_AWS_SAE1_PRDT_PRD_PRD_SERVICE_HOST=10.100.21.81
SHLVL=1
```

* The env var is `ERCASH_PARKINGLOT_SERVICE_AWS_SAE1_PRDT_PRD_PRD_PORT_80_TCP_ADDR`

2. Choose an API endpoint, say `/v2/parkinglots/14791/tickets/247803991853`, along with the set of headers to trigger the handlers in the service.
3. Execute with the wget command:

```console
$ kubectl -n supercash-aws-sae1-prdt-prd-prd run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -O- http://\${SUPERCASH_PARKINGLOT_SERVICE_AWS_SAE1_PRDT_PRD_PRD_PORT_80_TCP_ADDR}/v2/parkinglots/14791/tickets/247803991853\?scanning\=true --header=\"X-Supercash-App-Version: 1.0\" --header=\"X-Supercash-Marketplace-Id: 14824\" --header=\"User-Agent: IOS 15.0, iPhone 13 mini Apple\" --header=\"X-Supercash-Uid: 15241\" --header=\"X-Supercash-Store-Id: 14791\" --header=\"X-Supercash-Uid: 15241\" --header=\"X-Supercash-Tid: d5f465f2-a6c3-471f-b21c-1c21577e8e5a\"; done"
If you don't see a command prompt, try pressing enter.
wget: server returned error: HTTP/1.1 500
Connecting to 10.100.69.238 (10.100.69.238:80)
```

### K9s Terminal

```console
 Context: arn:aws:eks:sa-east-1:806101772216:cluster/eks-ppd-prdt-super-cash      <0> tail   <6> 1h   <shift-c> Clear               <t> Toggle Timesta… ____  __.________
 Cluster: arn:aws:eks:sa-east-1:806101772216:cluster/eks-ppd-prdt-super-cash      <1> head            <c>       Copy                <w> Toggle Wrap    |    |/ _/   __   \______
 User:    arn:aws:eks:sa-east-1:806101772216:cluster/eks-ppd-prdt-super-cash      <2> 1m              <m>       Mark                                   |      < \____    /  ___/
 K9s Rev: v0.25.18                                                                <3> 5m              <ctrl-s>  Save                                   |    |  \   /    /\___ \
 K8s Rev: v1.21.5-eks-bc4871b                                                     <4> 15m             <s>       Toggle AutoScroll                      |____|__ \ /____//____  >
 CPU:     2%                                                                      <5> 30m             <f>       Toggle FullScreen                              \/            \/
 MEM:     53%
┌──────────────────────── Logs(supercash-aws-sae1-prdt-prd-prd/supercash-parkinglot-service-aws-sae1-prdt-prd-prd-7d9d68fw5pdv:parkinglot-service)[1m] ─────────────────────────┐
│                                                       Autoscroll:On     FullScreen:Off     Timestamps:Off     Wrap:Off                                                        │
│ 2022-04-21T00:36:47 DEBUG [app=parkinglots-service,prof=observed,prd,db][tid=48773527a5439fb7,sid=48773527a5439fb7,sxp=false][uid=] 17 --- [nio-8082-exec-4] o.s.w.s.m.m.a.Ht │
│ 2022-04-21T00:36:47 DEBUG [app=parkinglots-service,prof=observed,prd,db][tid=48773527a5439fb7,sid=48773527a5439fb7,sxp=false][uid=] 17 --- [nio-8082-exec-4] o.s.web.servlet. │
│ [ACCESS] 172.16.2.4 [21/Apr/2022:00:36:47 +0000] 172.16.2.4 http_method=GET http_path=/parkinglots/actuator/health http_query= http_protocol=HTTP/1.1 http_status=200 latency │
│ 2022-04-21T00:36:47 DEBUG [app=parkinglots-service,prof=observed,prd,db][tid=,sid=,sxp=][uid=] 17 --- [gistrationTask1] o.s.web.client.RestTemplate              : HTTP POST  │
│ 2022-04-21T00:36:47 DEBUG [app=parkinglots-service,prof=observed,prd,db][tid=,sid=,sxp=][uid=] 17 --- [gistrationTask1] o.s.web.client.RestTemplate              : Accept=[ap │
│ 2022-04-21T00:36:47 DEBUG [app=parkinglot
```

### HPA Get status

1. NO traffic has no memory usage:

```console
$ kubectl get hpa -n supercash-aws-sae1-prdt-prd-prd
NAME                                                 REFERENCE                                                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
supercash-parkinglot-service-aws-sae1-prdt-prd-prd   Deployment/supercash-parkinglot-service-aws-sae1-prdt-prd-prd   0%/75%    1         4         1          35m
```

2. Once the traffic starts running, the metrics start channing... The status of the `TARGETS` show the values.

```console
$ kubectl get hpa -n supercash-aws-sae1-prdt-prd-prd
NAME                                                 REFERENCE                                                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
supercash-parkinglot-service-aws-sae1-prdt-prd-prd   Deployment/supercash-parkinglot-service-aws-sae1-prdt-prd-prd   32%/75%   1         4         1          28m
```

3. As it stays constant, you can now stop the service
  * The `TARGETS` will gradually go back to the cold state at `0%`.

```console
$ kubectl get hpa -n supercash-aws-sae1-prdt-prd-prd
NAME                                                 REFERENCE                                                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
supercash-parkinglot-service-aws-sae1-prdt-prd-prd   Deployment/supercash-parkinglot-service-aws-sae1-prdt-prd-prd   14%/75%   1         4         1          32m

$ kubectl get hpa -n supercash-aws-sae1-prdt-prd-prd
NAME                                                 REFERENCE                                                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
supercash-parkinglot-service-aws-sae1-prdt-prd-prd   Deployment/supercash-parkinglot-service-aws-sae1-prdt-prd-prd   14%/75%   1         4         1          32m

$ kubectl get hpa -n supercash-aws-sae1-prdt-prd-prd
NAME                                                 REFERENCE                                                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
supercash-parkinglot-service-aws-sae1-prdt-prd-prd   Deployment/supercash-parkinglot-service-aws-sae1-prdt-prd-prd   0%/75%    1         4         1          35m
```

## Adjusting HPA

* We need to know all models to understand the behavior of the service in order to scale it properly.
* The smaller the pod the higher the density as it can scale higher, with more pods, squeezed for more traffic.

### Too Low Memory

* With too low memory, the container is killed with `OOMKilled`.
  * Problem: That indicates the memory allocation is too low and the service can't even bootstrap to have its healthcheck passed!
  * Solution: Increase the resources values and make sure the container starts at minimum

```console

│ supercash-aws-sae1-prdt-prd-prd  supercash-parkinglot-service-aws-sae1-prdt-prd-prd-9754859rkt7g  ●  0/1          1 OOMKilled    0   5      0      0      5      2 172.16.3.7 │

```
