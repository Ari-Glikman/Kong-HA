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

<img width="1105" height="400" alt="image" src="https://github.com/user-attachments/assets/221a30a2-ce12-41e0-8829-6501ba8f70fd" />


| Pros | Cons |
|---|---|
| Deployment flexibility: Users can deploy groups of Data Planes in different data centers, geographies, or zones without needing a local clustered database for each DP group. | When running in traditional mode, every Kong Gateway node runs as both a Control Plane (CP) and Data Plane (DP). This means that if any of your nodes are compromised, the entire running gateway configuration is compromised.|
|  | If you’re running Kong Gateway Enterprise with Kong Manager, request throughput may be reduced on nodes running Kong Manager due to expensive calculations being run to render analytics data and graphs. |

To run this, in the [Traditional](https://github.com/Ari-Glikman/Kong-HA/tree/main/Deployment%20Topologies/Traditional) folder:
Make sure to set your environment variable IRIS_PASSWORD and review the YAML to make sure it points at your correct ports and hosts to get the IAM license.
```
docker compose up -d
```

Access the Kong Managers at http://localhost:8002 and http://localhost:8012.
Note that configurations set on one Manager are synced to the other as they share the same database.
Create some services and routes and test it out at exposed ports 8000 and 8010.

2) [Hybrid Mode](https://developer.konghq.com/gateway/hybrid-mode/)

<img width="978" height="1118" alt="image" src="https://github.com/user-attachments/assets/2e173b53-82d1-45f8-818a-afe4fa057e20" />

| Pros | Cons |
|---|---|
| Traditional mode is the only deployment topology that supports plugins that require a database, like rate limiting with the cluster strategy, or OAuth2. | No plugins that require a database, like rate limiting with the cluster strategy, or OAuth2. |
| Increased reliability: The availability of the database doesn’t affect the availability of the Data Planes. Each DP caches the latest configuration it received from the Control Plane on local disk storage, so if CP nodes are down, the DP nodes keep functioning. While the CP is down, DP nodes constantly try to reestablish communication. DP nodes can be restarted while the CP is down, and still proxy traffic normally. | |
| Traffic reduction: Drastically reduces the amount of traffic to and from the database, since only CP nodes need a direct connection to the database. ||
| Increased security: If one of the DP nodes is compromised, an attacker won’t be able to affect other nodes in the Kong Gateway cluster. ||
| Ease of management: Admins only need to interact with the CP nodes to control and monitor the status of the entire Kong Gateway cluster. ||

To run this, in the Hybrid folder:
Make sure to set your environment variable IRIS_PASSWORD and review the YAML to make sure it points at your correct ports and hosts to get the IAM license.

Note that for the first time we have introduced a certificate and key. For the sake of simplicity I will try to avoid them in these examples and write about how to properly set up secure communication in an upcoming InterSystems Developer Community article. That being said, Hybrid Mode requires that the CP and DP communicate via mTLS hence it will be necessary here.

Create a /Certificates folder in the CP directory that will be referenced in the volumes section of the YAML and in this directory run the following (assuming you are on powershell with OpenSSL 1.1.1+) to create the certificate and key:
```
openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:P-256 -nodes -sha256 -days 825 `
  -keyout admin.key -out admin.crt `
  -subj "/CN=kong_clustering" `
  -addext "subjectAltName=DNS:kong_clustering"
```
Make sure to copy and paste the certificates directory to also appear in the DP directory.

Once that is done you can go ahead and first for the CP and then for the DP:
```
docker compose up -d
```
You can now go and create some services and routes at the Kong Manager on http://localhost:8002 and test API requests out at exposed ports 8000 and 8100.

3) [DB-less Mode](https://developer.konghq.com/gateway/db-less-mode/)

<img width="1100" height="303" alt="image" src="https://github.com/user-attachments/assets/8e333062-a1c1-4a70-b461-dc6257cdec9f" />


| Pros | Cons |
|---|---|
| Reduced number of dependencies: No need to manage a database installation if the entire setup for your use-cases fits in memory. | No plugins that require a database, like rate limiting with the cluster strategy, or OAuth2. |
| Automation in CI/CD scenarios: Configuration for entities can be kept in a single source of truth managed via a Git repository. | |
| Enables more deployment options for Kong Gateway. ||

To run this, in the DB-less folder:
Make sure to set your environment variable IRIS_PASSWORD and review the YAML to make sure it points at your correct ports and hosts to get the IAM license.
```
docker compose up -d
```
Test from a REST Client such as Postman sending:

GET http://localhost:8000/somepath/ or
GET http://localhost:8010/somepath/

with the proper Authorization IRIS/HSHC expects.
