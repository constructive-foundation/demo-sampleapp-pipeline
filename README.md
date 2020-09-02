# demo-sampleapp-pipeline

Purpose of this repository is to build an image which runs a sample war file and release via helm.

## Building utility image

In order to run docker in docker or aws cli to authenticate with ecr or releaseing chart using helm a utility image is to be built.
It can be done as a separate process and can be stored in ECR. For example purpose, it has been considered that it is available post below command exection.

```bash
docker build -f Dockerfile.dind-aws -t dind-aws-helm:latest --build-arg VERSION=2.16.9 .
```

## Pre-requisites & assumption

In order to bake image for app, there are pre-reqs for Jenkins pipeline.

- ECR repository is already created
- AWS credentials are available as Global ENV variable, can be set vars using crendentials plugin
- tomcat.apache.org should be accessible.
- Pipeline has configuration of PAT for Github repository clone.
- Helm 2.16.9 version is chosen

## Jenkins server app

In order to release Jenkins on eks cluster, [jenkins-k8s](helm-charts/jenkins-k8s/Chart.yaml) chart can be released. It is required that tiller service account should have sifficient privilages to run the workload and rbac for resources across namespace.

```bash 
kubectl create sa tiller -n kube-system && kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
cd helm-charts/jenkins-k8s
helm init --upgrade --wait --skip-refresh --service-account tiller 
helm upgrade jenkins --namespace jenkins --install ."
```