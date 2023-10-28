```bash
GH_PROJ="https://github.com/okd-project/okd"
OKD_VERSION_AAA="4.12.0-0.okd-2023-03-18-084815"

wget "${GH_PROJ}/releases/download/${OKD_VERSION_AAA}/openshift-install-linux-${OKD_VERSION_AAA}.tar.gz"
```

```bash
gcloud projects list
gcloud config set project

export GCP_PROJECT=XXXX
export GCP_SA=bbb-okd
export GCP_DOMAIN=bbb.seems.cloud

gcloud config set compute/region europe-central2
gcloud config set compute/zone europe-central2-b

gcloud config list

gcloud services enable compute.googleapis.com --project ${GCP_PROJECT}
gcloud services enable cloudapis.googleapis.com --project ${GCP_PROJECT}
gcloud services enable cloudresourcemanager.googleapis.com --project ${GCP_PROJECT}
gcloud services enable dns.googleapis.com --project ${GCP_PROJECT}
gcloud services enable iamcredentials.googleapis.com --project ${GCP_PROJECT}
gcloud services enable iam.googleapis.com --project ${GCP_PROJECT}
gcloud services enable servicemanagement.googleapis.com --project ${GCP_PROJECT}
gcloud services enable serviceusage.googleapis.com --project ${GCP_PROJECT}
gcloud services enable storage-api.googleapis.com --project ${GCP_PROJECT}
gcloud services enable storage-component.googleapis.com --project ${GCP_PROJECT}

gcloud iam service-accounts create ${GCP_SA}

gcloud projects add-iam-policy-binding ${GCP_PROJECT} \
  --member "serviceAccount:${GCP_SA}@${GCP_PROJECT}.iam.gserviceaccount.com" \
  --role "roles/owner"
```

```bash
openshift-install create cluster --dir install_dir

openshift-install wait-for bootstrap-complete --dir install_dir
openshift-install wait-for install-complete --dir install_dir

openshift-install destroy cluster --dir install_dir
```

```yaml
apiVersion: v1
baseDomain: bbb.seems.cloud
credentialsMode: Mint
controlPlane:
  hyperthreading: Enabled
  name: master
  architecture: amd64
  platform:
    gcp:
      type: n2d-standard-8
      zones:
        - europe-central2-b
      osDisk:
        diskType: pd-ssd
        diskSizeGB: 192
      tags:
        - okd
        - okd-master
        - all
  replicas: 3
compute:
  - hyperthreading: Enabled
    name: worker
    architecture: amd64
    platform:
      gcp:
        type: n2d-standard-8
        zones:
          - europe-central2-b
        osDisk:
          diskType: pd-ssd
          diskSizeGB: 192
        tags:
          - okd
          - okd-worker
          - all
    replicas: 2
metadata:
  name: okd
networking:
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  machineNetwork:
    - cidr: 10.100.100.0/27
    - cidr: 10.200.200.0/23
  networkType: OVNKubernetes
  serviceNetwork:
    - 172.30.0.0/16
platform:
  gcp:
    projectID: core-337701
    region: europe-central2
    network: okd
    controlPlaneSubnet: control-plane
    computeSubnet: compute
    defaultMachinePlatform:
      tags:
        - okd
        - all
pullSecret: '{"auths": {"fake":{"auth":"fake"}}}'
sshKey: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxMR6x6rHclUBQI3IQVPZN8xjkAVVAZmS1PV/hNg/XPc5sl5fI7/3FLmwu+A9PDuiPu5++60Ns4NtYJcd+hVQ9m/htl6DGPeUoflin1pmVFSfKMUctTRWsl+e2ldt3CVmTgFclLABdLDR+cSb3jSqNXgonzjNcWbTfhrsSnqNuD+2GxXpGZXc4rYDloSrlGVOx0mEiyJrMocJFuVlh1JB8Os0KNnx5qD56h5zIRLGkhHhgXIO5kJ+hNB+vF3FV2Fq9Ar47+DrQiD/o9/h17HFDvD0tzze1GLYAJs4QcFJJPKdWM1kHyXa/p9TIFLc3rVnCrVx1NihgaEhiY+d452otV0p1Bq1tvotfPJ92BDSNlF7A1YuJNYqRkNNpwSPMznPQtVkeRCHTNH5MmMqhPptGEPLiDlkMUZeFFjTKz0IDo6QCX05WBl+SXYLo1l2R9jCoKstmoKvlsFY6fcYpSYj78X9E4bIX++LSLiG9oGXwg2xoZTlwofhqjnI+xc9tMiU=
```
