            +-------------+
            |   Start     |
            +------+------+  
                   |      
                   v
  +----------------+----------------+
  | Initialize Container           |
  +----------------+----------------+
                   |
                   v
     +-------------+--------------+
     | Command Execution          |
     +-------------+--------------+
                   |
                   v
+------------------+-------------------+
| Define Functions:                    |
| - safeWriteSecret                    |
+------------------+-------------------+
                   |
                   v
   +---------------+----------------+
   | Create Config                  |
   +---------------+----------------+
                   |
                   v
   +---------------+----------------+
   | Create Bootnodes ConfigMap     |
   +---------------+----------------+
                   |
                   v
   +---------------+----------------+
   | Create Genesis ConfigMap       |
   +---------------+----------------+
                   |
                   v
   +---------------+----------------+
   | Create Static-Nodes JSON       |
   +---------------+----------------+
                   |
                   v
   +---------------+----------------+
   | Loop through Validators        |
   +---------------+----------------+
                   |
                   v
+------------------+--------------------+
| 1. Generate Validator Keys and Details|
+------------------+--------------------+
                   |
                   v
+------------------+--------------------+
| a. Generate Node Keys                 |
|   +----------------+----------------+ |
|   | i. Generate private key:        | |
|   |    - nodekey                    | |
|   +----------------+----------------+ |
|   | ii. Generate public key:        | |
|   |    - nodekey.pub                | |
|   +----------------+----------------+ |
|   | Command:                        | |
|   | openssl genpkey -algorithm RSA  | |
|   | -out nodekey                    | |
|   +----------------+----------------+ |
|   | openssl rsa -in nodekey         | |
|   | -pubout -out nodekey.pub        | |
|   +----------------+----------------+ |
                   |
                   v
+------------------+--------------------+
| b. Generate Account Details           |
|   +----------------+----------------+ |
|   | i. Generate private key:        | |
|   |    - accountPrivateKey          | |
|   +----------------+----------------+ |
|   | ii. Generate password:          | |
|   |    - accountPassword            | |
|   +----------------+----------------+ |
|   | iii. Generate keystore file:    | |
|   |    - accountKeystore            | |
|   +----------------+----------------+ |
|   | iv. Generate address:           | |
|   |    - accountAddress             | |
|   +----------------+----------------+ |
|   | Command:                        | |
|   | echo "PrivateKey" > accountPrivateKey |
|   | echo "Password" > accountPassword |
|   | echo "Keystore" > accountKeystore |
|   | echo "Address" > accountAddress  | |
|   +----------------+----------------+ |
                   |
                   v
+------------------+--------------------+
| 2. Store Keys in Kubernetes Secret    |
+------------------+--------------------+
                   |
                   v
+------------------+--------------------+
| a. Execute safeWriteSecret Function   |
|   +----------------+----------------+ |
|   | i. Create Kubernetes secret      | |
|   |    with generated keys and files | |
|   |    Command:                      | |
|   | kubectl create secret generic    | |
|   | ${key}-keys --namespace <NS>     | |
|   | --from-file=nodekey=<path>/nodekey| |
|   | --from-file=nodekey.pub=<path>/nodekey.pub |
|   | --from-file=enode=<path>/nodekey.pub |
|   | --from-file=accountPrivateKey=<path>/accountPrivateKey |
|   | --from-file=accountPassword=<path>/accountPassword |
|   | --from-file=accountKeystore=<path>/accountKeystore |
|   | --from-file=accountAddress=<path>/accountAddress |
|   +----------------+----------------+ |
                   |
                   v
+------------------+--------------------+
| 3. Create ConfigMap for Validator Address |
+------------------+--------------------+
                   |
                   v
+------------------+--------------------+
| a. Create ConfigMap                  |
|   +----------------+----------------+ |
|   | i. Command:                      | |
|   | kubectl create configmap         | |
|   | --namespace <NS> besu-node-validator-${i}-address |
|   | --from-file=address=<path>/accountAddress |
|   +----------------+----------------+ |
                   |
                   v
+------------------+--------------------+
| 4. Update Static-Nodes JSON           |
+------------------+--------------------+
                   |
                   v
+------------------+--------------------+
| a. Add Validator Public Key to JSON   |
|   +----------------+----------------+ |
|   | i. Extract public key:            | |
|   | pubkey=$(cat <path>/nodekey.pub)  | |
|   +----------------+----------------+ |
|   | ii. Add to static-nodes.json:    | |
|   | echo ",\"enode://$pubkey@besu-node-validator-$i-0.besu-node-validator-$i.<NS>.svc.cluster.local:30303?discport=0\"" >> <path>/static-nodes.json |
|   +----------------+----------------+ |
                   |
                   v
+------------------+--------------------+
| 5. Finalize Static-Nodes JSON         |
+------------------+--------------------+
                   |
                   v
+------------------+--------------------+
| a. Remove Extra Comma                 |
|   +----------------+----------------+ |
|   | i. Command:                      | |
|   | sed -i '0,/,/s///' <path>/static-nodes.json |
|   +----------------+----------------+ |
                   |
                   v
+------------------+--------------------+
| 6. Write Peers ConfigMap              |
+------------------+--------------------+
                   |
                   v
+------------------+--------------------+
| a. Execute safeWriteBesuPeersConfigmap|
|   +----------------+----------------+ |
|   | i. Command:                      | |
|   | kubectl get configmap --namespace <NS> besu-peers |
|   +----------------+----------------+ |
|   | ii. If not exists, create:       | |
|   | kubectl create configmap --namespace <NS> besu-peers --from-file=static-nodes.json=<path>/static-nodes.json |
|   +----------------+----------------+ |
                   |
                   v
              +----+----+
              | Completed|
              +---------+


---
The `consensys/quorum-k8s-hooks` image is a Docker image provided by ConsenSys designed to facilitate the deployment and management of Quorum blockchain nodes within a Kubernetes environment. It contains essential scripts and tools needed for initializing and managing the blockchain network, particularly focusing on the genesis block generation and initial configuration of the nodes.

#### Key Features and Benefits
1. **Automated Genesis Block Generation**:
    - The image includes tools like `quorum-genesis-tool` which automates the creation of the genesis block. This ensures that the blockchain starts with a well-defined initial state tailored to the network's configuration.

2. **Secure Secrets Management**:
    - The image provides functions for securely writing secrets to cloud providers' secret management services (e.g., Azure Key Vault, AWS Secrets Manager) or Kubernetes secrets. This ensures that sensitive information, such as node keys and account private keys, is handled securely.

3. **Kubernetes Integration**:
    - The image is optimized for Kubernetes, with scripts that handle the creation and management of Kubernetes ConfigMaps and secrets. This tight integration simplifies the deployment process and ensures that all necessary configurations are available to the blockchain nodes.

4. **Multi-Cloud Support**:
    - The image supports deployment across different cloud providers, including Azure and AWS, as well as local environments. This flexibility allows organizations to choose their preferred cloud provider without changing their deployment scripts significantly.

#### Usage in Kubernetes Job
The `consensys/quorum-k8s-hooks` image is used in a Kubernetes Job definition to:
1. **Generate the Genesis Configuration**:
    - The Job runs a container from this image to generate the genesis configuration and other necessary blockchain files using `quorum-genesis-tool`.
    - Parameters such as consensus algorithm, chain ID, block period, and more are specified to tailor the genesis block to the network's requirements.

2. **Create and Manage ConfigMaps and Secrets**:
    - The Job script includes functions (`safeWriteSecret`, `safeWriteBesuPeersConfigmap`, and `safeWriteGenesisConfigmap`) to create and update Kubernetes ConfigMaps and secrets. These functions ensure that the configuration files and keys are securely stored and accessible to the blockchain nodes.

3. **Bootstrap Validator Nodes**:
    - The script handles the setup of validator nodes by generating keys and storing them securely. It also updates the `static-nodes.json` file to include the newly generated validator nodes, ensuring they can communicate and participate in consensus.

---
