
<img width="647" height="491" alt="image" src="https://github.com/user-attachments/assets/c296c824-b94f-442e-9322-d231ddf3a960" />


# Argo Setting Up


## 1. install docker
### Add Docker Repository

```sh
sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc


echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


sudo systemctl enable --now docker
sudo docker run --rm hello-world
```



## 2- Install kubectl
kubectl is the command-line tool used to manage Kubernetes clusters.

```sh
K8S_VERSION="$(curl -L -s https://dl.k8s.io/release/stable.txt)"

curl -LO "https://dl.k8s.io/release/${K8S_VERSION}/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl

kubectl version --client
```



## 3- Install Kind
Download and install the Kind binary:

```sh
KIND_VERSION="$(curl -s https://api.github.com/repos/kubernetes-sigs/kind/releases/latest | grep '"tag_name"' | cut -d '"' -f 4)"

### download kind from K8s
curl -Lo kind "https://kind.sigs.k8s.io/dl/${KIND_VERSION}/kind-linux-amd64"

### download kind from github
curl -fL --connect-timeout 15 -o kind \
  https://github.com/kubernetes-sigs/kind/releases/download/v0.33.0/kind-linux-amd64


chmod +x kind
sudo install kind /usr/local/bin/kind
rm kind

kind version
docker version
```


## 4- Create a Kubernetes cluster
Create a configuration file:

vi kind-config.yaml
``sh
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
```

Save the file, then create the cluster:
kind create cluster --name lab --config kind-config.yaml
```

Kind downloads the Kubernetes node image and starts the cluster using Docker containers. The process can take several minutes.
- Verify your Kubernetes cluster
- Check the cluster context:

```sh
kubectl config get-contexts

kubectl config current-context

kubectl get nodes -o wide

kubectl get pods -A

kubectl cluster-info --context kind-lab
```


## 5- install helm

### Link = https://helm.sh/docs/intro/install/

From Apt (Debian/Ubuntu)​
Members of the Helm community have contributed an Apt package for Debian/Ubuntu. This package is generally up to date. Thanks to Buildkite for hosting the repo.

```sh
HELM_BUILDKITE_APT_KEY_ID="DDF78C3E6EBB2D2CC223C95C62BA89D07698DBC6"

sudo apt-get install curl gpg apt-transport-https --yes

curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey > "${TMPDIR:-/tmp}/helm.gpg"

# Ensure that the key ID matches to prevent a repository compromise from establishing an attacker controlled key
if [ "$(gpg --show-keys --with-colons "${TMPDIR:-/tmp}/helm.gpg" | awk -F: '$1 == "fpr" {print $10}' | head -n 1)" != "${HELM_BUILDKITE_APT_KEY_ID}" ]; then echo "ERROR: Unexpected Helm APT key ID: potential key compromise"; exit 1; fi

cat "${TMPDIR:-/tmp}/helm.gpg" | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

sudo apt-get update
sudo apt-get install helm
```



## 6- install NGINX Ingress Controller
first check the cluster:

```sh
kubectl get nodes

kind get clusters

helm repo list

###  Install the NGINX Ingress Controller using its Helm chart:
helm repo add ingress-nginx  https://kubernetes.github.io/ingress-nginx

helm repo update

kubectl create namespace ingress-nginx

helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx

helm ls -A
```


## 7-Install ArgoCD 

###Link:
https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd
https://github.com/argoproj/


