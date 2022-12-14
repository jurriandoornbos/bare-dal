# z2jh-dal
Jupyterhub Kubernetes setup for the DAL
## Client side:

and connect to the server:



## Server side setup:
#setup containers

 gcloud container clusters create \
  --machine-type e2-standard-2 \
  --num-nodes 1 \
  --zone europe-west4-b\
  --cluster-version latest \
 k8s-test

#set admin roles
  kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=jurriandoornbos@gmail.com

 #User node-pool that can downscale (limit to 3 users (as each users is essentially the size of a node))
 gcloud beta container node-pools create user-pool \
  --machine-type n2d-standard-8 \
  --num-nodes 0 \
  --enable-autoscaling \
  --min-nodes 0 \
  --max-nodes 3 \
  --node-labels hub.jupyter.org/node-purpose=user \
  --node-taints hub.jupyter.org_dedicated=user:NoSchedule \
  --zone europe-west4-b \
  --cluster k8s-test


#install helm

#add helm repo jhub
 helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
 helm repo update

# setup the cluster using the helm chart

 helm upgrade --cleanup-on-fail \
  --install dal9000 jupyterhub/jupyterhub \
  --namespace dal9000 \
  --create-namespace \
  --version=2.0.0 \
  --values config.yaml

#create random ssl hex

 openssl rand -hex 32
2c80123964734554ec91b4dc23eda0c0f974983c55eadfc36bad1755c74bde67

#twice
28d34b5c4c86f08d710403df53acb63f76dbc31981772f34a00790e96c59e39e

#kill everything:
 gcloud container clusters delete k8s-tester --zone=europe-west4-b


## switch kubectl clusters
gcloud container clusters get-credentials k8s-test \
 --zone europe-west4-b