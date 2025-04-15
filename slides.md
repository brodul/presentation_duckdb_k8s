
## Use duckDb to analyze k8s clusters
### Andraz Brodnik - brodul

---

## What is Kubernetes? ☸

Scheduler with an CRUD API ...

- Create
- Read
- Update
- Delete

with an HTTP API call

---

## How do we answer complex questions? 

- How many STS pods are in an AZ (AWS) or a Zone (GCP)?
- Can we delete the node without disrupting customer in namespace `duckinc`

---

# Meet DuckDb 🦆

---

## Simple 4 step solution 📓

---

## Step 1 - Get the data

`kubectl get pods -A` 

---

<img alt="image" src="https://github.com/user-attachments/assets/70711567-fc53-4146-9da2-34a9e77c4d3e" />

---

## How do we read k8s data? 

`YAML` or `json`

```shell
kubectl get pods -A -o json > pods.json
kubectl get nodes -o json > nodes.json
```
---
#### nodes.json
```json
{
  "apiVersion": "v1",
  "kind": "List",
  "items": [
    {
      "apiVersion": "v1",
      "kind": "Node",
      "metadata": {
        "name": "node-a1",
        "labels": {
          "topology.kubernetes.io/zone": "us-central1-a"
        }
      }
    },
```
---
#### pods.json
```json
{
  "apiVersion": "v1",
  "kind": "List",
  "items": [
    {
      "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
        "name": "web-server-1",
        "namespace": "production"
      },
      "spec": {
        "nodeName": "node-a1"
      },
```
---

## Step 2 - Flatten the data 🔨

list of simple object ( values are not objects)

```shell
jq '
  .items
  | map({
      podName: .metadata.name,
      namespace: .metadata.namespace,
      nodeName: .spec.nodeName
    })
' < pods.json > flat_pods.json
```
---
```json
jq '
  .items
  | map({
      az: .metadata.labels."topology.kubernetes.io/zone",
      nodeName: .metadata.name
    })
' < nodes.json > flat_nodes.json
```
---
#### flat_nodes.json
```json
[
  {
    "az": "us-central1-a",
    "nodeName": "node-a1"
  },
  {
    "az": "us-central1-b",
    "nodeName": "node-b1"
  }
]
```
---
#### flat_pods.json
```json
[
  {
    "podName": "web-server-1",
    "namespace": "production",
    "nodeName": "node-a1"
  },
  {
    "podName": "api-service-1",
    "namespace": "production",
    "nodeName": "node-b1"
  },
```
---

## Step 3 - Create a table

like in a releational database

```shell
duckdb my.db
```
---
```sql
CREATE TABLE pods AS
SELECT * FROM read_json_auto('flat_pods.json');
CREATE TABLE nodes AS
SELECT * FROM read_json_auto('flat_nodes.json');
```
---
```
D SELECT * FROM pods;
┌───────────────┬────────────┬──────────┐
│    podName    │ namespace  │ nodeName │
│    varchar    │  varchar   │ varchar  │
├───────────────┼────────────┼──────────┤
│ web-server-1  │ production │ node-a1  │
│ api-service-1 │ production │ node-b1  │
│ db-1          │ database   │ node-b1  │
│ cache-1       │ cache      │ node-b1  │
│ job-runner-1  │ batch-jobs │ node-a1  │
└───────────────┴────────────┴──────────┘
```
---
```
D SELECT * FROM nodes;
┌───────────────┬──────────┐
│      az       │ nodeName │
│    varchar    │ varchar  │
├───────────────┼──────────┤
│ us-central1-a │ node-a1  │
│ us-central1-b │ node-b1  │
└───────────────┴──────────┘
```
---
## Step 4 - Join and query tables for the answer 🧪

... with ChatGpt 

---
```sql
SELECT 
    p.podName,
    p.namespace,
    p.nodeName,
    n.az
FROM 
    pods p
JOIN 
    nodes n ON p.nodeName = n.nodeName;
```
---
```
┌───────────────┬────────────┬──────────┬───────────────┐
│    podName    │ namespace  │ nodeName │      az       │
│    varchar    │  varchar   │ varchar  │    varchar    │
├───────────────┼────────────┼──────────┼───────────────┤
│ web-server-1  │ production │ node-a1  │ us-central1-a │
│ api-service-1 │ production │ node-b1  │ us-central1-b │
│ db-1          │ database   │ node-b1  │ us-central1-b │
│ cache-1       │ cache      │ node-b1  │ us-central1-b │
│ job-runner-1  │ batch-jobs │ node-a1  │ us-central1-a │
└───────────────┴────────────┴──────────┴───────────────┘
```
---
#### All of this will persist in your file `my.db`
---

<img  alt="image" src="https://github.com/user-attachments/assets/8937adcc-e072-47db-9104-907ce8ef1b02" />

