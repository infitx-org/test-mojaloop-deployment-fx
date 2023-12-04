## Mojaloop Switch and Mojaloop Connectorts Deployment on K8S to test FX

### System configuration
* Ubuntu 23.04 x86_64
* AMD Ryzen 9 3900X 12-Core Processor
* 32GB RAM
* 512GB nvme storage


### Deploying K8S

Followed the instructions provided in this link
https://docs.mojaloop.io/legacy/deployment-guide/local-setup-linux.html

- sudo snap install microk8s --classic --channel=1.27/stable
- add the user vijay to the 'microk8s' group
  ```
  sudo usermod -a -G microk8s $USER
  sudo chown -f -R $USER ~/.kube
  ```
-
  ```
  microk8s.kubectl get nodes
  microk8s.enable ingress dns helm3
  sudo snap alias microk8s.kubectl kubectl
  sudo snap alias microk8s.helm3 helm
  ```
- Get k8s config file using `microk8s config` to access it from remotely


### Setup Helm repositories

- Dependencies
    ```
    helm repo add stable https://charts.helm.sh/stable
    helm repo add incubator https://charts.helm.sh/incubator
    helm repo add kiwigrid https://kiwigrid.github.io
    helm repo add kokuwa https://kokuwaio.github.io/helm-charts
    helm repo add elastic https://helm.elastic.co
    helm repo add codecentric https://codecentric.github.io/helm-charts
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo add mojaloop-charts https://mojaloop.github.io/charts/repo
    helm repo add redpanda-console https://packages.vectorized.io/public/console/helm/charts/
    ```
- Mojaloop Chart
    ```
    helm repo add mojaloop https://mojaloop.io/helm/repo/
    ```
- `helm repo update`


### Deploy default Mojaloop

- Deploy backend dependencies like mysql, kafka...etc
  ```
  helm --namespace demo install backend mojaloop/example-mojaloop-backend
  ```
- Deploy mojaloop
  ```
  helm --namespace demo install moja mojaloop/mojaloop --version 15.2.0 --create-namespace
  ```
- Wait for all pods to be healthy
- Run Helm Test
  ```
  helm -n demo test moja
  ```

### Customize mojaloop values file for FX functionality

- Download the default values file of mojaloop chart https://github.com/mojaloop/helm/blob/v15.2.0/mojaloop/values.yaml
- Change central-ledger version from `v17.3.2` to `v17.4.0-snapshot.10`
- Upgrade mojaloop with new values file
  ```
  helm --namespace demo upgrade moja mojaloop/mojaloop --version 15.2.0 -f ~/Work/mojaloop/test-mojaloop-deployment-fx/values-mojaloop.yaml
  ```

### Deploy FX participants

  ```
  cd participants-fx-poc
  sh build.sh
  helm --namespace demo install fx .
  ```