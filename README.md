# Bank of Anthos

![GitHub branch check runs](https://img.shields.io/github/check-runs/GoogleCloudPlatform/bank-of-anthos/main)
[![Website](https://img.shields.io/website?url=https%3A%2F%2Fcymbal-bank.fsi.cymbal.dev%2F&label=live%20demo
)](https://cymbal-bank.fsi.cymbal.dev)

**Bank of Anthos** is a sample HTTP-based web app developed at Google that simulates a bank's payment processing network, allowing users to create artificial bank accounts and complete transactions.

This application works on any Kubernetes cluster.

## Screenshots

| Sign in                                                                                                        | Home                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Login](/docs/img/login.png)](/docs/img/login.png) | [![User Transactions](/docs/img/transactions.png)](/docs/img/transactions.png) |


## Service architecture

![Architecture Diagram](/docs/img/architecture.png)

| Service                                                 | Language      | Description                                                                                                                                  |
| ------------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/src/frontend)                              | Python        | Exposes an HTTP server to serve the website. Contains login page, signup page, and home page.                                                |
| [ledger-writer](/src/ledger/ledgerwriter)              | Java          | Accepts and validates incoming transactions before writing them to the ledger.                                                               |
| [balance-reader](/src/ledger/balancereader)            | Java          | Provides efficient readable cache of user balances, as read from `ledger-db`.                                                                |
| [transaction-history](/src/ledger/transactionhistory)  | Java          | Provides efficient readable cache of past transactions, as read from `ledger-db`.                                                            |
| [ledger-db](/src/ledger/ledger-db)                     | PostgreSQL    | Ledger of all transactions. Option to pre-populate with transactions for demo users.                                                         |
| [user-service](/src/accounts/userservice)              | Python        | Manages user accounts and authentication. Signs JWTs used for authentication by other services.                                              |
| [contacts](/src/accounts/contacts)                     | Python        | Stores list of other accounts associated with a user. Used for drop down in "Send Payment" and "Deposit" forms.                              |
| [accounts-db](/src/accounts/accounts-db)               | PostgreSQL    | Database for user accounts and associated data. Option to pre-populate with demo users.                                                      |
| [loadgenerator](/src/loadgenerator)                    | Python/Locust | Continuously sends requests imitating users to the frontend. Periodically creates new accounts and simulates transactions between them.      |

## Quickstart (GKE)

1. Clone the repository.

   ```sh
   git clone https://github.com/falcon646/bank-of-anthos
   cd bank-of-anthos/
   ```

2. Deploy Bank of Anthos to the cluster.

   ```sh
   kubectl apply -f ./extras/jwt/jwt-secret.yaml
   kubectl apply -f ./kubernetes-manifests
   ```

3. Wait for the pods to be ready.

   ```sh
   kubectl get pods
   ```

   After a few minutes, you should see the Pods in a `Running` state:

   ```
   NAME                                  READY   STATUS    RESTARTS   AGE
   accounts-db-6f589464bc-6r7b7          1/1     Running   0          99s
   balancereader-797bf6d7c5-8xvp6        1/1     Running   0          99s
   contacts-769c4fb556-25pg2             1/1     Running   0          98s
   frontend-7c96b54f6b-zkdbz             1/1     Running   0          98s
   ledger-db-5b78474d4f-p6xcb            1/1     Running   0          98s
   ledgerwriter-84bf44b95d-65mqf         1/1     Running   0          97s
   loadgenerator-559667b6ff-4zsvb        1/1     Running   0          97s
   transactionhistory-5569754896-z94cn   1/1     Running   0          97s
   userservice-78dc876bff-pdhtl          1/1     Running   0          96s
   ```

7. Access the web frontend in a browser using the frontend's external IP.

   ```sh
   kubectl get service frontend | awk '{print $4}'
   ```

   Visit `http://EXTERNAL_IP` in a web browser to access your instance of Bank of Anthos.

## Additional deployment options

- **Workload Identity**: [See these instructions.](/docs/workload-identity.md)
- **Cloud SQL**: [See these instructions](/extras/cloudsql) to replace the in-cluster databases with hosted Google Cloud SQL.
- **Multi Cluster with Cloud SQL**: [See these instructions](/extras/cloudsql-multicluster) to replicate the app across two regions using GKE, Multi Cluster Ingress, and Google Cloud SQL.
- **Istio**: [See these instructions](/extras/istio) to configure an IngressGateway.
- **Anthos Service Mesh**: ASM requires Workload Identity to be enabled in your GKE cluster. [See the workload identity instructions](/docs/workload-identity.md) to configure and deploy the app. Then, apply `extras/istio/` to your cluster to configure frontend ingress.
- **Java Monolith (VM)**: We provide a version of this app where the three Java microservices are coupled together into one monolithic service, which you can deploy inside a VM (eg. Google Compute Engine). See the [ledgermonolith](/src/ledgermonolith) directory.

## Documentation

<!-- This section is duplicated in the docs/ README: https://github.com/GoogleCloudPlatform/bank-of-anthos/blob/main/docs/README.md -->

- [Development](/docs/development.md) to learn how to run and develop this app locally.
- [Environments](/docs/environments.md) to learn how to deploy on non-GKE clusters.
- [Workload Identity](/docs/workload-identity.md) to learn how to set-up Workload Identity.
- [CI/CD pipeline](/docs/ci-cd-pipeline.md) to learn details about and how to set-up the CI/CD pipeline.
- [Troubleshooting](/docs/troubleshooting.md) to learn how to resolve common problems.

## Demos featuring Bank of Anthos
- [Tutorial: Explore Anthos (Google Cloud docs)](https://cloud.google.com/anthos/docs/tutorials/explore-anthos)
- [Tutorial: Migrating a monolith VM to GKE](https://cloud.google.com/migrate/containers/docs/migrating-monolith-vm-overview-setup)
- [Tutorial: Running distributed services on GKE private clusters using ASM](https://cloud.google.com/service-mesh/docs/distributed-services-private-clusters)
- [Tutorial: Run full-stack workloads at scale on GKE](https://cloud.google.com/kubernetes-engine/docs/tutorials/full-stack-scale)
- [Architecture: Anthos on bare metal](https://cloud.google.com/architecture/ara-anthos-on-bare-metal)
- [Architecture: Creating and deploying secured applications](https://cloud.google.com/architecture/security-foundations/creating-deploying-secured-apps)
- [Keynote @ Google Cloud Next '20: Building trust for speedy innovation](https://www.youtube.com/watch?v=7QR1z35h_yc)
- [Workshop @ IstioCon '22: Manage and secure distributed services with ASM](https://www.youtube.com/watch?v=--mPdAxovfE)
