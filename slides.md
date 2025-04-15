
## Use duckDb to analyze k8s clusters
### Andraz Brodnik - brodul

---

## What is Kubernetes? ☸

Scheduler with an CRUD API.

Where you Create, Read, Update and Delete the objects/manifest from/to the state saved somewhere

---

## How do we answer complex questions? 

- How many nodes are in an AZ (AWS) or Zones (GCP)?
- Which nodes has no pods that are in the `fizbuz` namespace?

---

# Meet DuckDb 🦆

---

# Simple 4 step solution 📓

---

# Step 1 - Get the data

list of objects (values can be objects)

---

<img width="70vw" alt="image" src="https://github.com/user-attachments/assets/70711567-fc53-4146-9da2-34a9e77c4d3e" />

---

## How do we read k8s data? 

`YAML` or `json`

```shell
kubectl get pods -o json > pods.json
kubectl get nodes -o json > nodes.json
```
---
```json
{
    "apiVersion": "v1",
    "kind": "Node",
    "metadata": {
        "annotations": {
            "volumes.kubernetes.io/controller-managed-attach-detach": "true"
        },
        "creationTimestamp": "2025-03-22T20:10:41Z",
        "labels": {
            "topology.kubernetes.io/zone": "us-central1-a",
            "kubernetes.io/os": "linux"
        },
        "name": "node01",
```
---
```json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "labels": {
            "app.kubernetes.io/instance": "my-release",
            "app.kubernetes.io/managed-by": "Helm",
            "app.kubernetes.io/name": "nginx",
            "app.kubernetes.io/version": "1.27.4",
            "helm.sh/chart": "nginx-19.1.0",
            "pod-template-hash": "756589d594"
        },
        "name": "my-release-nginx-756589d594-qtrfp",
        "namespace": "default",
        "nodeName": "node01",
        "preemptionPolicy": "PreemptLowerPriority",
        "priority": 0,
```
---

# Step 2 - Flatten the data 🔨

list of simple object ( values are not objects)

```shell
jq '.items | map( {podName: .metadata.name, namespace: .metadata.namespace, nodeName: .spec.nodeName } )' < pods.json > flat_pods.json
jq '.items | map( {az: .metadata.labels."topology.kubernetes.io/zone", nodeName: .metadata.name} )' < nodes.json > flat_nodes.json
```

---

# Step 3 - Create a table

like in a releational database

`duckdb my.db`

---
```sql
CREATE TABLE pods AS
SELECT * FROM read_json_auto('flat_pods.json');
CREATE TABLE nodes AS
SELECT * FROM read_json_auto('flat_nodes.json');
```
---

# Step 4 - Join and query tables for the answer 🧪

... with ChatGpt 

---

Thats it ... 
