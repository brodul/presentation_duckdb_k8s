
## Use duckDb to analyze k8s clusters
### Andraz Brodnik - brodul

---

## What is Kubernetes? ☸

Scheduler with an CRUD API.

Where you Create, Read, Update and Delete the objects/manifest from/to the state saved somewhere

---

## How do we read k8s data? 

`YAML` or `json`

`kubectl get pods -o json` 

`kubectl get nodes -o json`

---

## How do we ansver complex questions? 

- How many nodes are in an AZ (AWS) or Zones (GCP)?
- Which nodes has no pods that are in the `fizbuz` namespace

---

# Meet DuckDb 🦆

---

# Simple 4 step solution 📓

--- 

# Step 1 - Get the data

list of objects (values can be objects)

---

<img width="808" alt="image" src="https://github.com/user-attachments/assets/70711567-fc53-4146-9da2-34a9e77c4d3e" />

---

# Step 2 - Flatten the data ) 🔨

list of simple object ( values are not objects)

`jq '.items | map( {podName: .metadata.name, namespace: .metadata.namespace, nodeName: .spec.nodeName } )' < pods.json`
`jq '.items | map( {az: .metadata.labels."topology.kubernetes.io/zone", nodeName: .metadata.name} )' < nodes.json`
---

# Step 3 - Create a table ⎍

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

---
