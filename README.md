Kubernetes cluster with K3s
===============================================
Bootstrapping kubernetes cluster with k3s, opensuse, vagrant, virtualbox and deploy a go application


Prerequisites
--------------
Make sure you have installed all of the prerequisites on your development machine(macos):
* VirtualBox
```bash
brew install --cask virtualbox
```
* Vagrant
```bash
brew install -cask vagrant
```
* Docker - [Download & Install Docker](https://docs.docker.com/docker-for-mac/install/).


Build docker image and push on DockerHub
----------------------------------------
```bash
cd go-helloworld
docker build -t go-helloworld .
# replace [muktadiur] with your dockerhub prefix
docker tag go-helloworld [muktadiur]/go-helloworld:v1.0.0 

#login to docker(need account on dockerhub and credentails)
docker login
# replace [muktadiur] with your dockerhub prefix
docker push [muktadiur]/go-helloworld:v1.0.0
```

Provisioning Kubernetes cluster with K3s
----------------------------------------
```bash
vagrant up

vagrant ssh opensusevm1
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip=192.168.56.41 --flannel-iface=eth1" sh -
sudo su -
zypper install -t pattern apparmor
# get token
cat /var/lib/rancher/k3s/server/token

vagrant ssh opensusevm2
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip=192.168.56.42 --flannel-iface=eth1" \
K3S_URL="https://192.168.56.41:6443" K3S_TOKEN="xxxxxx::server:xxxxxxx" sh -
sudo su -
zypper install -t pattern apparmor

#on opensusevm1 to view nodes
sudo su -
kubectl get nodes
```

Deploy application on cluster
-----------------------------

```bash
vagrant ssh opensusevm1
sudo su -
kubectl create ns test
kubectl get ns
kubectl create deploy go-helloworld --image=muktadiur/go-helloworld:v1.0.0 -r=3 -n test
kubectl get deploy
kubectl expose deploy go-helloworld --port=6111 -n test
kubectl get svc
kubectl get svc go-helloworld -n test -o yaml > go-helloworld-nodeport.yaml
# Update type: ClusterIP to type: NodePort and add nodePort: 30020
kubectl apply -f go-helloworld-nodeport.yaml
# test the app. 
curl 192.168.56.41:30020
```

Sample go-helloworld-nodeport.yaml:

```bash
apiVersion: v1
kind: Service
metadata:
  labels:
    app: go-helloworld
  name: go-helloworld-nodeport
  namespace: test
spec:
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 6111
    protocol: TCP
    targetPort: 6111
    nodePort: 30020
  selector:
    app: go-helloworld
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

