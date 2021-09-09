# Go Demo

## Commands
Execute 'export KUBECONFIG=../devops24/cluster/kubecfg-eks' before running any kubectl commands

## Kubernetes Setup
* kubectl apply -f k8s/build-ns.yml --record
* kubectl apply -f k8s/prod-ns.yml --record
* kubectl apply -f k8s/jenkins.yml --record
