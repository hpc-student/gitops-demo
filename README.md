# Step-by-step instructions to recreate the setup

## create github personal token with the next permissions; and export "GITHUB_TOKEN" and "GITHUB_USER" as env variables :

   - <img src="./PAT_permissions.png" alt="PAT_permissions" width="500"/>



## fork both repos to your account :
### repo 1 (app + CI) : git@github.com:hpc-student/http_server.git :
- this includes the ruby-simple-server application
- Dockerfile 
- Github Actions workflow listen to push events > build new image and push it to registry
- as registry we are using GitHub Container Registry
- once you fork the repo the workflow will be trigered and a new image will be built and pushed to your github container registry "ghcr" ,to check it go to: your profile > packages 

### repo 2 ( CD with FluxV2 - GitOps ): git@github.com:hpc-student/gitops-demo.git
- this includes the main work
## we assume you have kubectl configured with your k8s cluster , in case of minikube enable "metallb","metrics-server" addons.

## install flux V2 CLI
> curl -s https://fluxcd.io/install.sh | sudo bash
>
> #check that you have everything
>
> flux check --pre

## bootstrap everything with :
> flux bootstrap github \
>
>  --components-extra=image-reflector-controller,image-automation-controller \
>
>  --owner=$GITHUB_USER \
>
>  --repository=gitops-demo \
>
>  --branch=main \
>
>  --path=./clusters/dev-cluster \
>
>  --read-write-key \
>
>  --personal

<hr>
<hr>
<hr>


# the Strategy/Architecture: (draft)
## we will use GitOps workflow with flux v2
<img src="strtg1.png" alt="image1" width="500"/>   >>>  <img src="./strtg2.png" alt="image2" width="500"/>

## CI

check the applictaion repo : https://github.com/hpc-student/http_server.git

it includes Dockerfile and  github actions workflow

when you push code to the main branch the workflow will build a new image and push it to "github container registry" and that's all for the CI

it doesn't need to triger the CD 

> if you like to keep k8s manifests with its application you can do that and then reference the app repo here but that's not what we are going to do
>
> since our developers never work on the infrastructure or the delivery of the application it is easier for Ops to centralize configurations in one repo and this what we are going to do


## CD

we will be using IaC
this repo will be the "single source of truth" it will be a mirror to what is going on our k8s clusters 

we will insatll apps and controllers by pushing yaml files to this repo and Flux will sync the modifications to the cluster

Flux will also listen to our container registry "github container registry" and will update the applictaion when there is a new image , Flux will commit changes to this repo before applying it on the cluster this way this repo will always be in sync with the cluster 

so even if you don't have access with kubectl you must be able to track modifications "like wich version of the app is running" just by checking this repo

### steps 
- setting the basic CI for the applications 
- using tags to track image versions

- setting basic CD by installing Flux
- create k8s manifests for the application : deployment - service ... etc
- setting Image Update Automation
- install traefic
- configure traefic 


# high availability 
in production multiple master plan on deffirent AZ

eks control plane

# kops
# metallb


#
- ingress controller 
- auto scaller
- grafana stack

#
create name spcae
flux create source helm traefik --url https://helm.traefik.io/traefik --namespace traefik
flux create helmrelease my-traefik --chart traefik \
  --source HelmRepository/traefik \
  --chart-version 10.0.0 \
  --namespace traefik \
  --export
# traefik
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" -n traefik --output=name) 9000:9000 -n traefik
http://0.0.0.0:9000/dashboard/

## minikube
minikube addons enable ingress

## hpa
enable metrics server


## Automate image updates to Git 

flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --owner=$GITHUB_USER \
  --repository=gitops-demo \
  --branch=main \
  --path=./clusters/dev-cluster \
  --read-write-key \
  --personal


  flux create image repository http-server \
--image=ghcr.io/hpc-student/http_server \
--interval=1m \
--export > ./clusters/dev-cluster/http-server/registry-http-server.yaml