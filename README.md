# Google Cloud Platform multi-zone deployment of 'echo server'

## Create GKE Autopilot cluster

### public cluster default parameters:

```sh
gcloud container clusters create-auto "autopilot-cluster-1" --project "{project-id}" \
--region "europe-west6" \
--release-channel "regular" \
--network "projects/{project-id}/global/networks/default" \
--subnetwork "projects/{project-id}/regions/europe-west6/subnetworks/default" \
--cluster-ipv4-cidr "/17" \
--services-ipv4-cidr "/22"
```

### public cluster with security group:

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

### private cluster:

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

## Create Load Balancer to support multi-region

[Google Cload: How to create Load Balancer](https://cloud.google.com/run/docs/multiple-regions#create-lb)

[Multi-cluster Services](https://cloud.google.com/kubernetes-engine/docs/concepts/multi-cluster-services)
