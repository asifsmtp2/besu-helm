# Quorum-Kubernetes (k8s)

The following repo has example reference implementations of private networks using k8s. These examples are aimed at developers and ops people to get them familiar with how to run a private ethereum network in k8s and understand the concepts involved.

You will need the following tools to proceed:

- [Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/) This is the local equivalent of a K8S cluster (refer to the [playground](./playground) for manifests to deploy)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/)
- [Helm Diff plugin](https://github.com/databus23/helm-diff)

Verify kubectl is connected with (please use the latest version of kubectl)
```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:18:22Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

Install helm & helm-diff:
Please note that the documentation and steps listed use *helm3*. The API has been updated so please take that into account if using an older version
```bash
$ helm plugin install https://github.com/databus23/helm-diff --version master
```

The repo provides examples using multiple tools such as kubectl, helm etc. Please select the one that meets your deployment requirements.

-----------
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-operator prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
-----------

besu-genesis/templates/: Templates for genesis block setup.
besu-node/templates/: Templates for deploying Besu nodes.
blockscout/charts/: Sub-charts for Blockscout dependencies.
blockscout/templates/: Templates for deploying Blockscout.
explorer/templates/: Templates for deploying a blockchain explorer.

kubectl create namespace besu

cd helm

helm install genesis ./charts/besu-genesis --namespace besu --create-namespace --values ./values/genesis-besu.yml

helm install validator-1 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-2 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-3 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-4 ./charts/besu-node --namespace besu --values ./values/validator.yml


helm install rpc-1 ./charts/besu-node --namespace besu --values ./values/reader.yml

----------------

kubectl get pods -n besu


kubectl get pods --all-namespaces

----------------

### Detailed Explanation

#### 1. Genesis Block Deployment
```bash
helm install genesis ./charts/besu-genesis --namespace besu --create-namespace --values ./values/genesis-besu.yml
```
- `helm install genesis`: This installs a Helm chart with the release name `genesis`.
- `./charts/besu-genesis`: Specifies the path to the Helm chart for deploying the genesis block.
- `--namespace besu`: Deploys the release in the `besu` namespace. If the namespace does not exist, it will be created.
- `--create-namespace`: Creates the `besu` namespace if it doesn't already exist.
- `--values ./values/genesis-besu.yml`: Uses the values file `genesis-besu.yml` for configuration.

Purpose: This command initializes the Besu blockchain network by setting up the genesis block, which contains the initial state and configuration of the network.

### 2. Bootnodes Deployment (Optional but Recommended)
Bootnodes are special nodes that help other nodes discover each other in the network.

```bash
helm install bootnode-1 ./charts/besu-node --namespace besu --values ./values/bootnode.yml
helm install bootnode-2 ./charts/besu-node --namespace besu --values ./values/bootnode.yml
```
- `helm install bootnode-1`: Installs a Helm chart with the release name `bootnode-1`.
- `./charts/besu-node`: Specifies the path to the Helm chart for deploying a Besu node.
- `--namespace besu`: Deploys the release in the `besu` namespace.
- `--values ./values/bootnode.yml`: Uses the values file `bootnode.yml` for configuration.

Purpose: These commands deploy two bootnodes (`bootnode-1` and `bootnode-2`) that help other nodes in the network to find and connect to each other.

### 3. Validators Deployment
Validators are responsible for validating transactions and creating new blocks in the blockchain.

```bash
helm install validator-1 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-2 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-3 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-4 ./charts/besu-node --namespace besu --values ./values/validator.yml
```
- `helm install validator-1`: Installs a Helm chart with the release name `validator-1`.
- `./charts/besu-node`: Specifies the path to the Helm chart for deploying a Besu node.
- `--namespace besu`: Deploys the release in the `besu` namespace.
- `--values ./values/validator.yml`: Uses the values file `validator.yml` for configuration.

Purpose: These commands deploy four validator nodes (`validator-1`, `validator-2`, `validator-3`, `validator-4`) that will participate in the consensus process to validate transactions and create new blocks.

### 4. Besu and Tessera Node Pair Deployment
Deploys a Besu node paired with a Tessera node for privacy transactions.

```bash
helm install member-1 ./charts/besu-node --namespace besu --values ./values/txnode.yml
```
- `helm install member-1`: Installs a Helm chart with the release name `member-1`.
- `./charts/besu-node`: Specifies the path to the Helm chart for deploying a Besu node.
- `--namespace besu`: Deploys the release in the `besu` namespace.
- `--values ./values/txnode.yml`: Uses the values file `txnode.yml` for configuration.

Purpose: This command deploys a node (`member-1`) that includes both Besu and Tessera for handling privacy-enabled transactions.

### 5. RPC Node Deployment
Deploys an RPC node that provides an API for interacting with the blockchain.

```bash
helm install rpc-1 ./charts/besu-node --namespace besu --values ./values/reader.yml
```
- `helm install rpc-1`: Installs a Helm chart with the release name `rpc-1`.
- `./charts/besu-node`: Specifies the path to the Helm chart for deploying a Besu node.
- `--namespace besu`: Deploys the release in the `besu` namespace.
- `--values ./values/reader.yml`: Uses the values file `reader.yml` for configuration.

Purpose: This command deploys an RPC node (`rpc-1`) that provides JSON-RPC and GraphQL APIs for interacting with the blockchain.

----

The provided documentation gives detailed instructions on deploying various components of a Quorum or Besu blockchain network using Helm charts. Here’s an explanation of the key sections and commands:

### General Configuration

#### 1. Cluster Configuration
This section specifies general cluster settings, including the cloud provider and whether to use cloud-native services.

```yaml
cluster:
  provider: local  # choose from: local | aws | azure
  cloudNativeServices: false # set to true to use Cloud Native Services (SecretsManager and IAM for AWS; KeyVault & Managed Identities for Azure)
```

- `provider`: Specifies the environment where the cluster is deployed. Options are `local`, `aws`, or `azure`.
- `cloudNativeServices`: When set to `true`, it enables the use of cloud-native services for managing secrets and identities.

#### 2. AWS and Azure Configuration
These sections provide specific settings for deploying to AWS or Azure environments.

AWS:
```yaml
aws:
  serviceAccountName: quorum-sa
  region: ap-southeast-2
```

Azure:
```yaml
azure:
  identityName: quorum-pod-identity
  identityClientId: azure-clientId
  keyvaultName: azure-keyvault
  tenantId: azure-tenantId
  subscriptionId: azure-subscriptionId
```

- `serviceAccountName`: The name of the service account used for deploying resources.
- `region`: The region where resources will be deployed.
- `identityClientId`, `keyvaultName`, `tenantId`, `subscriptionId`: Specific settings for using Azure managed identities and Key Vault.

### Setting Up a Local Development Environment

#### Using Minikube
For local development, Minikube is used to create a local Kubernetes cluster.

```bash
minikube start --memory 16384 --cpus 2
# or with RBAC
minikube start --memory 16384 --cpus 2 --extra-config=apiserver.Authorization.Mode=RBAC

# enable the ingress
minikube addons enable ingress

# optionally start the dashboard
minikube dashboard &
```

- Memory and CPU Allocation: It is recommended to allocate at least 16GB of memory and 2 CPUs for running Besu nodes.
- RBAC: Role-Based Access Control can be enabled for added security.
- Ingress and Dashboard: Enable the ingress addon and optionally start the Kubernetes dashboard for easier management.

#### Verifying kubectl Connection
Ensure kubectl is connected to the Minikube cluster.

```bash
kubectl version
```

### Usage Instructions

#### Deploying ELK Stack for Logs
ElasticSearch, Kibana, and Filebeat can be deployed for log management.

```bash
helm repo add elastic https://helm.elastic.co
helm repo update
# if on cloud
helm install elasticsearch --version 7.17.1 elastic/elasticsearch --namespace quorum --create-namespace --values ./values/elasticsearch.yml
# if local - set the replicas to 1
helm install elasticsearch --version 7.17.1 elastic/elasticsearch --namespace quorum --create-namespace --values ./values/elasticsearch.yml --set replicas=1 --set minimumMasterNodes=1
helm install kibana --version 7.17.1 elastic/kibana --namespace quorum --values ./values/kibana.yml
helm install filebeat --version 7.17.1 elastic/filebeat  --namespace quorum --values ./values/filebeat.yml
```

- Helm Repositories: Add the Elastic Helm repository and update.
- Install ElasticSearch, Kibana, and Filebeat: Install these components, customizing values as needed, especially for local development.

#### Deploying Prometheus Stack for Metrics
Prometheus and Grafana can be deployed for monitoring.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack --version 34.10.0 --namespace=quorum --create-namespace --values ./values/monitoring.yml --wait
kubectl --namespace quorum apply -f  ./values/monitoring/
```

- Helm Repositories: Add the Prometheus community Helm repository and update.
- Install Prometheus Stack: Install the kube-prometheus-stack, configuring alerts and other settings via `monitoring.yml`.

#### Deploying Blockchain Explorer (Blockscout)
Blockscout can be deployed to explore the blockchain.

```bash
helm dependency update ./charts/blockscout

# For GoQuorum
helm install blockscout ./charts/blockscout --namespace quorum --values ./values/blockscout-goquorum.yml

# For Besu
helm install blockscout ./charts/blockscout --namespace quorum --values ./values/blockscout-besu.yml
```

- Update Dependencies: Ensure Helm dependencies are up to date.
- Install Blockscout: Deploy Blockscout for either GoQuorum or Besu using the appropriate values file.

#### Deploying Quorum Explorer
Quorum Explorer can be deployed for network management and exploration.

```bash
helm install quorum-explorer ./charts/explorer --namespace quorum --create-namespace --values ./values/explorer-besu.yaml
```

- Install Quorum Explorer: Deploy the Quorum Explorer, ensuring configuration details are updated post-deployment if necessary.

### Example Helm Commands for Deploying Besu and GoQuorum Nodes



#### For Besu:
```bash
helm install genesis ./charts/besu-genesis --namespace besu --create-namespace --values ./values/genesis-besu.yml

# Bootnodes (optional but recommended)
helm install bootnode-1 ./charts/besu-node --namespace besu --values ./values/bootnode.yml
helm install bootnode-2 ./charts/besu-node --namespace besu --values ./values/bootnode.yml

# Validators
helm install validator-1 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-2 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-3 ./charts/besu-node --namespace besu --values ./values/validator.yml
helm install validator-4 ./charts/besu-node --namespace besu --values ./values/validator.yml

# Besu and Tessera node pair
helm install member-1 ./charts/besu-node --namespace besu --values ./values/txnode.yml

# RPC node
helm install rpc-1 ./charts/besu-node --namespace besu --values ./values/reader.yml
```

#### For GoQuorum:
```bash
helm install genesis ./charts/goquorum-genesis --namespace quorum --create-namespace --values ./values/genesis-goquorum.yml

# Validators
helm install validator-1 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml
helm install validator-2 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml
helm install validator-3 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml
helm install validator-4 ./charts/goquorum-node --namespace quorum --values ./values/validator.yml

# Quorum and Tessera node pair
helm install member-1 ./charts/goquorum-node --namespace quorum --values ./values/txnode.yml

# RPC node
helm install rpc-1 ./charts/goquorum-node --namespace quorum --values ./values/reader.yml
```

### Ingress Setup for External Access

#### Deploy Ingress Controller
Set up an ingress controller for handling external traffic.

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install quorum-network-ingress ingress-nginx/ingress-nginx \
    --namespace quorum \
    --set controller.ingressClassResource.name="network-nginx" \
    --set controller.ingressClassResource.controllerValue="k8s.io/network-ingress-nginx" \
    --set controller.replicaCount=1 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.service.externalTrafficPolicy=Local

kubectl apply -f ../ingress/ingress-rules-besu.yml
```

### API Access
Examples of how to interact with the blockchain using HTTP RPC and GraphQL APIs.

#### HTTP RPC API:
```bash
curl -v -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://<INGRESS_IP>/rpc
```

#### HTTP GraphQL API:
```bash
curl -X POST -H "Content-Type: application/json" --data '{ "query": "{syncing{startingBlock currentBlock highestBlock}}"}' http://<INGRESS_IP>/graphql/
```

### Summary
This documentation provides comprehensive instructions for deploying and managing a blockchain network using Helm charts, including local development setup, cloud deployment configurations, and optional components like ELK for logging and Prometheus for metrics. Experimentation and customization are encouraged to fit specific requirements and policies.
