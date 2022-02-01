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
