```bash
GH_PROJ="https://github.com/okd-project/okd"

OKD_VERSION_AAA="4.11.0-0.okd-2022-12-02-145640"

wget "${GH_PROJ}/releases/download/${OKD_VERSION_AAA}/openshift-install-linux-${OKD_VERSION_AAA}.tar.gz"
wget "${GH_PROJ}/releases/download/${OKD_VERSION_AAA}/openshift-client-linux-${OKD_VERSION_AAA}.tar.gz"
```

```bash
GCP_PROJ="core-337701"
GCP_SA=bbb-okd
GCP_DOMAIN=bbb.seems.cloud

echo -e "Project:\t\t${GCP_PROJ}\nService Account:\t${GCP_SA}\nDomain:\t\t\t${GCP_DOMAIN}"

gcloud config set project "${GCP_PROJ}"
gcloud config set compute/region europe-central2
gcloud config set compute/zone europe-central2-b

gcloud config list

gcloud services enable compute.googleapis.com --project ${GCP_PROJ}
gcloud services enable cloudapis.googleapis.com --project ${GCP_PROJ}
gcloud services enable cloudresourcemanager.googleapis.com --project ${GCP_PROJ}
gcloud services enable dns.googleapis.com --project ${GCP_PROJ}
gcloud services enable iamcredentials.googleapis.com --project ${GCP_PROJ}
gcloud services enable iam.googleapis.com --project ${GCP_PROJ}
gcloud services enable servicemanagement.googleapis.com --project ${GCP_PROJ}
gcloud services enable serviceusage.googleapis.com --project ${GCP_PROJ}
gcloud services enable storage-api.googleapis.com --project ${GCP_PROJ}
gcloud services enable storage-component.googleapis.com --project ${GCP_PROJ}

gcloud iam service-accounts create ${GCP_SA}

gcloud projects add-iam-policy-binding ${GCP_PROJ} \
  --member "serviceAccount:${GCP_SA}@${GCP_PROJ}.iam.gserviceaccount.com" \
  --role "roles/owner"
```

```bash
mkdir -p install_dir/

cat > install_dir/install-config.yaml << "EndOfMessage"
apiVersion: v1
baseDomain: bbb.seems.cloud
credentialsMode: Mint
metadata:
  name: test-okd
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
        - test-okd
        - test-okd-control-plane
        - test-okd-master
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
            - test-okd
            - test-okd-compute
            - test-okd-worker
            - all
    replicas: 2
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
pullSecret: '{"auths": {"fake":{"auth":"fake"}}}'
sshKey: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxMR6x6rHclUBQI3IQVPZN8xjkAVVAZmS1PV/hNg/XPc5sl5fI7/3FLmwu+A9PDuiPu5++60Ns4NtYJcd+hVQ9m/htl6DGPeUoflin1pmVFSfKMUctTRWsl+e2ldt3CVmTgFclLABdLDR+cSb3jSqNXgonzjNcWbTfhrsSnqNuD+2GxXpGZXc4rYDloSrlGVOx0mEiyJrMocJFuVlh1JB8Os0KNnx5qD56h5zIRLGkhHhgXIO5kJ+hNB+vF3FV2Fq9Ar47+DrQiD/o9/h17HFDvD0tzze1GLYAJs4QcFJJPKdWM1kHyXa/p9TIFLc3rVnCrVx1NihgaEhiY+d452otV0p1Bq1tvotfPJ92BDSNlF7A1YuJNYqRkNNpwSPMznPQtVkeRCHTNH5MmMqhPptGEPLiDlkMUZeFFjTKz0IDo6QCX05WBl+SXYLo1l2R9jCoKstmoKvlsFY6fcYpSYj78X9E4bIX++LSLiG9oGXwg2xoZTlwofhqjnI+xc9tMiU=
EndOfMessage
```

```bash
openshift-install create install-config --dir install_dir
openshift-install create cluster --dir install_dir

openshift-install wait-for bootstrap-complete --dir install_dir
openshift-install wait-for install-complete --dir install_dir

openshift-install destroy cluster --dir install_dir
```
