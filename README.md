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
    --project={project_id}
```

### Deploy GKE Autopilot Cluster

Deploy two k8s clusters in different regions in specified GoogleCloud project:

```sh
gcloud container clusters create-auto echo-au \
    --region=australia-southeast2 \
    --release-channel=stable \
    --enable-private-nodes \
    --project={project_id}

gcloud container clusters create-auto echo-jp \
    --region=asia-northeast3 \
    --release-channel=stable \
    --enable-private-nodes \
    --project={project_id}
```

### Get credentials to kubectl configs

Setup kubectl client context configs for specified clusters:

```sh 
gcloud container clusters get-credentials echo-au \
    --region=australia-southeast2 \
    --project={project_id}

gcloud container clusters get-credentials echo-jp \
    --region=asia-northeast3 \
    --project={project_id}
```

### Register clusters to a fleet

```sh
gcloud container hub memberships register echo-au \
    --gke-cluster australia-southeast2/echo-au \
    --enable-workload-identity \
    --project={project_id}

gcloud container hub memberships register echo-jp \
    --gke-cluster asia-northeast3/echo-jp \
    --enable-workload-identity \
    --project={project_id}
```

### List registered clusters

```sh
gcloud container hub memberships list --project={project_id}
```

### Specify a config cluster

https://cloud.google.com/kubernetes-engine/docs/how-to/multi-cluster-ingress-setup#specifying_a_config_cluster


```sh
gcloud beta container hub ingress enable \
   --config-membership=echo-au
```

### Describe multi-cluster ingress

```sh
gcloud beta container hub ingress describe
```

### Configure Docker credentials for Artifact Registry

```sh
gcloud auth configure-docker \
    australia-southeast2-docker.pkg.dev
```

### Push images to Artifact Registry

```sh
docker pull ealen/echo-server
docker tag ealen/echo-server australia-southeast2-docker.pkg.dev/{project_id}/docker-registry/ealen/echo-server
docker push australia-southeast2-docker.pkg.dev/{project_id}/docker-registry/ealen/echo-server
```
## TODO: Configure image repository

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
