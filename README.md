# Google Cloud Platform multi-cluster deployment of 'echo server'

## Create GKE Autopilot multi-cluster

[Multi-cluster Ingress setup](https://cloud.google.com/kubernetes-engine/docs/how-to/multi-cluster-ingress-setup)

[Multi-cluster Services](https://cloud.google.com/kubernetes-engine/docs/concepts/multi-cluster-services)

### Enable services neccessary for Multi Cluster Ingress

with Anthos

```sh
gcloud services enable \
    anthos.googleapis.com \
    multiclusteringress.googleapis.com \
    gkehub.googleapis.com \
    container.googleapis.com \
    --project=project_id
```

standalone (without Anthos)

```sh
gcloud services enable \
    multiclusteringress.googleapis.com \
    gkehub.googleapis.com \
    container.googleapis.com \
    --project=project_id
```

### Deploy GKE Cluster

```sh
gcloud container clusters create-auto echo-au \
    --region=australia-southeast2 \
    --workload-pool=project_id.svc.id.goog \
    --release-channel=stable \
    --enable-private-nodes \
    --project=project_id

gcloud container clusters create-auto echo-au \
    --region=asia-northeast3 \
    --workload-pool=project_id.svc.id.goog \
    --release-channel=stable \
    --enable-private-nodes \
    --project=project_id
```

### Create public cluster with default parameters:

```sh
gcloud container clusters create-auto "autopilot-cluster-1" --project "{project-id}" \
--region "europe-west6" \
--release-channel "regular" \
--network "projects/{project-id}/global/networks/default" \
--subnetwork "projects/{project-id}/regions/europe-west6/subnetworks/default" \
--cluster-ipv4-cidr "/17" \
--services-ipv4-cidr "/22"
```

### Create public cluster with security group:

```sh
gcloud container clusters create-auto "autopilot-cluster-1" --project "{project-id}" \
--region "europe-west6" \
--release-channel "regular" \
--network "projects/{project-id}/global/networks/default" \
--subnetwork "projects/{project-id}/regions/europe-west6/subnetworks/default" \
--cluster-ipv4-cidr "/17" \
--services-ipv4-cidr "/22" \
--security-group "gke-security-groups@{group-name}"
```

### Create private cluster:

```sh
gcloud container clusters create-auto "autopilot-cluster-1" \
--project "{project-id}" \
--region "europe-west6" \
--release-channel "regular" \
--network "projects/{project-id}/global/networks/default" \
--subnetwork "projects/{project-id}/regions/europe-west6/subnetworks/default" \
--cluster-ipv4-cidr "/17" --services-ipv4-cidr "/22" \
--enable-private-nodes
```

### network routing setup:

```sh
gcloud compute routers create {NAT_ROUTER} \
--network {NETWORK} \
--region {REGION} \
--project={PROJECT_ID}

gcloud compute routers nats create NAT_CONFIG \
--region REGION \
--router NAT_ROUTER \
--nat-all-subnet-ip-ranges \
--auto-allocate-nat-external-ips \
--project=PROJECT_ID
```

### get cluster credentials to connect cluster

```sh
gcloud container clusters get-credentials CLUSTER_NAME \
    --region REGION \
    --project=PROJECT_ID
```

### Delete all resources in namespace

```sh
kubectl delete all --all -n {namespace}
```
