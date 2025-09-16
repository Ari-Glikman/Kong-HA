# Kong High Availability Deployments via InterSystems API Manager

The purpose of this repo is to provide some concrete docker-compose.yml examples to help you experiment with different Kong deployment topologies. 
**This is not meant to teach you how Kong/IAM works, rather to help you understand how to deploy a Highly Available IAM once you have basic understanding of IAM. To learn how IAM works complete the pre-requisites.**

Pre-Requisites:

* You should make sure to first complete [Kong EE "Zero to Hero" training repo](https://github.com/grongierisc/kong-ee-training). 
* Review [Installing IAM 3.10](https://docs.intersystems.com/components/csp/docbook/DocBook.UI.Page.cls?KEY=CIAM3.10_install)
* Make sure you have a valid IAM license.
* You will need to have an IRIS/HSHC installation running that IAM can connect to.

## IAM Deployment Topologies

IAM (essentially Kong Gateway) has 3 [deployment topologies](https://developer.konghq.com/gateway/deployment-topologies/) that allow for HA:

1) [Kong Traditional Mode](https://developer.konghq.com/gateway/traditional-mode/): [Multiple Node Clusters](https://developer.konghq.com/gateway/traditional-mode/#multiple-node-clusters)

<img width="1099" height="401" alt="image" src="https://github.com/user-attachments/assets/155ac60e-8d6a-4505-b2d7-ca3aa077c3b2" />

| Pros | Cons |
|---|---|
| Deployment flexibility: Users can deploy groups of Data Planes in different data centers, geographies, or zones without needing a local clustered database for each DP group. | When running in traditional mode, every Kong Gateway node runs as both a Control Plane (CP) and Data Plane (DP). This means that if any of your nodes are compromised, the entire running gateway configuration is compromised.|
|  | If you’re running Kong Gateway Enterprise with Kong Manager, request throughput may be reduced on nodes running Kong Manager due to expensive calculations being run to render analytics data and graphs. |

2) [Hybrid Mode](https://developer.konghq.com/gateway/hybrid-mode/)

<img width="954" height="1126" alt="image" src="https://github.com/user-attachments/assets/1545c401-6350-4ffb-b638-d68fcdd5e54b" />

| Pros | Cons |
|---|---|
| Traditional mode is the only deployment topology that supports plugins that require a database, like rate limiting with the cluster strategy, or OAuth2. | No plugins that require a database, like rate limiting with the cluster strategy, or OAuth2. |
| Increased reliability: The availability of the database doesn’t affect the availability of the Data Planes. Each DP caches the latest configuration it received from the Control Plane on local disk storage, so if CP nodes are down, the DP nodes keep functioning. While the CP is down, DP nodes constantly try to reestablish communication. DP nodes can be restarted while the CP is down, and still proxy traffic normally. | |
| Traffic reduction: Drastically reduces the amount of traffic to and from the database, since only CP nodes need a direct connection to the database. ||
| Increased security: If one of the DP nodes is compromised, an attacker won’t be able to affect other nodes in the Kong Gateway cluster. ||
| Ease of management: Admins only need to interact with the CP nodes to control and monitor the status of the entire Kong Gateway cluster. ||

3) [DB-less Mode](https://developer.konghq.com/gateway/db-less-mode/)

<img width="1084" height="303" alt="image" src="https://github.com/user-attachments/assets/6a6400b6-2eed-4de1-8681-f219c948ad59" />


| Pros | Cons |
|---|---|
| Reduced number of dependencies: No need to manage a database installation if the entire setup for your use-cases fits in memory. | No plugins that require a database, like rate limiting with the cluster strategy, or OAuth2. |
| Automation in CI/CD scenarios: Configuration for entities can be kept in a single source of truth managed via a Git repository. | |
| Enables more deployment options for Kong Gateway. ||
