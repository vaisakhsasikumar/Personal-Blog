---
layout: post
title:  "Deploying an internal container registry"
description: Using Digital Ocean Kubernetes Platform to Build an Internal Container Registry 
date:   2021-01-06 10:03:36 +0530
categories: containers
---


This Post includes Deploying an internal container registry using Digitalocean Kubernetes and Harbor. This is one of the best ways to get started to learn about Containers, Helm and Harbor

## Here are few articles to have a TL-DR about Kubernetes

[Kubernetes Tutorial](https://kubernetes.io/docs/tutorials/
)


[Kubernetes on Digitalocean](https://www.digitalocean.com/community/tech_talks/getting-started-with-kubernetes-on-digitalocean
)


[Deploy your app on Digitalocean](https://www.digitalocean.com/community/tech_talks/how-to-deploy-your-application-or-microservice-as-a-kubernetes-deployment
)


[Deploying Microservices on Kubernetes](https://www.digitalocean.com/community/tech_talks/deploying-microservices-as-kubernetes-daemonsets-and-jobs
)


[How to use Kubernetes to Expose your app](https://www.digitalocean.com/community/tech_talks/how-to-use-kubernetes-services-to-expose-your-app
)



## Creating a Kubernetes Cluster


![Configuration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ks7dxizrpuchmj3j7g5o.png)

![Digitalocean allows easy Deployement for Kubernetes](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jg2sww4p1t3co30wfab4.png)


Once Deployed you can download the k8s Configuration yaml file


![Kubernetes Dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b9emq7c11dm93pkuuynm.png)


After creating a cluster, you need to add an authentication token or certificate to your kubectl configuration file to connect.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xiqtyd157skl9j6vbc6p.png)

## Helm Package Manager

Helm Package manager is similar to Pip and Brew, and acts as a package manager to install different packages in Kubernetes. I have added the steps to install helm.

[How to install Software on Helm](https://www.digitalocean.com/community/tutorials/how-to-install-software-on-kubernetes-clusters-with-the-helm-3-package-manager)

To Install Harbor you need to have Helm package manager.

Use this script `wget https://raw.githubusercontent.com/farhankn/kube_challenge/main/helm.sh` to install helm and Harbor with the Default Credential

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7oq1opvpc5zf4kwzf9m5.png)


## Install Trefik Ingress
This allows you to fix the issue were external IP was not getting assigned after we  deploy the Harbor instance using Helm. I spend a considerable amount of time trying to troubleshoot how to fix including changing the expose type from ingress to nodeport and loadbalance.

```
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik
```


## Install Harbor using Helm

Before we install harbor using helm , we need to define some of the values in a [yaml file](https://github.com/farhankn/kube_challenge/blob/main/harbor.yml) . I’ve mentioned the type as ingress and disabled TLS for simplicity of configuration. We need to specify the url and Credentialsthat will be used to access the application in the file.
Once the yaml file has been created we can go ahead installing using helm with below commands.

```
helm repo add harbor https://helm.goharbor.io
helm repo update
helm install my-release harbor/harbor -f harbor.yaml
```

Once harbor install is complete , it can be access from URL specified in the harbor.yaml file. The below screenshot shows my deployed instance of harbor



## Installation Successful for Harbor
 
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o8pl8kxjkdc3sue30q0r.png)
 
