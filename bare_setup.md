# ddal working on baremetal server



https://georgepaw.medium.com/jupyterhub-with-kubernetes-on-single-bare-metal-instance-tutorial-67cbd5ec0b00
https://github.com/rohinijoshi06/jupyterhub-on-k8s

0. Install microk8s snap
https://microk8s.io/docs/getting-started


1. enable dns 
2. enable openebs
3. enable metallb

4. apply storage yaml

5. apply metallb yaml


15. install jhub

#add helm repo jhub
 helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
 helm repo update



# setup the cluster using the helm chart

 microk8s helm upgrade --cleanup-on-fail \
  --install dal9000 jupyterhub/jupyterhub \
  --namespace dal9000 \
  --create-namespace \
  --version=2.0.0 \
  --values config.yaml


kubectl --namespace dal9000 get pods

16. check internal ip
curl -i $(kubectl --namespace jhub get service | grep proxy-public | awk '{print $3}')

kubectl --namespace jhub get service


17. setup load-balancer
nano metal_config.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - PUBLIC_IP_ADDRESS-PUBLIC_IP_ADDRESS

kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml

kubectl apply -f metal_config.yaml


kubectl --namespace=metallb-system get pods

kubectl logs -l component=speaker -n metallb-system


kubectl --namespace jhub get service

18. reverse proxy nginx
nano default

# Default server configuration
#

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name _;
    location / {
        proxy_pass http://IP_ADDRESS_HERE/;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    access_log /root/logs/nginx-access.log;
    error_log /root/logs/nginx-error.log;

}

19. install nginx
sudo apt-get update
sudo apt-get -y install nginx
sudo systemctl restart nginx
mkdir /root/logs
cp default /etc/nginx/sites-available/default
sudo nginx -t
sudo systemctl reload nginx

20. then it works!


21. teardown:
helm delete jhub --purge
kubectl delete namespace jhub
helm delete pgdatabase --purge
kubectl delete namespace pgdatabase
kubectl drain $(hostname) --delete-local-data --force --ignore-daemonsets
kubectl delete node $(hostname)
kubeadm reset -f
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

