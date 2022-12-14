## Server side setup:
#setup containers

 gcloud container clusters create-auto \
  --region europe-west4\
  --cluster-version latest\
 k8s-auto



## setup gcloud connection to kubectl k8s-auto instead of k8s-tester
gcloud container clusters get-credentials k8s-auto \
 --region europe-west4

# check if it worked
gcloud container clusters describe k8s-auto \
 --region europe-west4

 # set admin roles
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=jurriandoornbos@gmail.com



#install helm

#add helm repo jhub
 helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
 helm repo update

 # setup the cluster using the helm chart

 helm upgrade --cleanup-on-fail \
  --install auto9000 jupyterhub/jupyterhub \
  --namespace auto9000 \
  --create-namespace \
  --version=2.0.0 \
  --timeout 10m0s \
  --values config_auto.yaml

# check status

kubectl -n auto9000 describe svc

kubectl -n auto900 describe pods

kubectl -n auto9000 get pods
kubectl -n auto9000 get svc

kubectl -n auto9000 logs hub-

kubectl get deployments
kubectl drain NODEn --ignore-daemonsets --force

# DESTROY EVERYTHING
gcloud container clusters delete k8s-auto --region=europe-west4