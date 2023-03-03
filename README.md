# RHODF

# Table of Contents 

- [Prerequisites](#Prerequisites)
- [kustomization.yaml](#kustomization.yml)
- [namespace-openshift-storage.yaml](#namespace-openshift-storage.yaml)
- [operatorgroup-openshift-storage-operatorgroup.yaml](#operatorgroup-openshift-storage-operatorgroup.yaml)
- [subscription-odf-operator.yaml](#subscription-odf-operator.yaml)
- [storagecluster-ocs-storagecluster.yaml](#storagecluster-ocs-storagecluster.yaml)
- [Tips](#Tips)

## Red Hat OpenShift Data Foundation

Red Hat OpenShift Data Foundation is software-defined storage that is optimized for container environments. It runs as an operator on OpenShift Container Platform to provide highly integrated and simplified persistent storage management for containers.

Provides the following storage classes to Red Hat OpenShift:

- ocs-storagecluster-ceph-rbd - Block storage
- ocs-storagecluster-cephfs - Shared filesystem
- ocs-storagecluster-ceph-rgw - S3 object storage.  Provides Object Bucket Claims (OBCs) using the RGW
- openshift-storage.noobaa.io - S3 object storage. Provides Object Bucket Claims (OBCs) using Multi-Cloud Gateway (MCG)

Some Links:

- Product Documentation for Red Hat OpenShift Data Foundation - https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation
- GitHub - https://github.com/red-hat-storage/odf-operator
- Red Hat Openshift Data Foundation Customer Portal Articles - https://access.redhat.com/taxonomy/products/red-hat-openshift-data-foundation
- Red Hat OpenShift Data Foundation RSS feed - https://access.redhat.com/term/Red%20Hat%20OpenShift%20Data%20Foundation/feed
- What is supported in Red Hat OpenShift Data Foundation (previously known as OpenShift Container Storage) 4.X? - https://access.redhat.com/articles/5001441
- Red Hat Bugzilla - https://bugzilla.redhat.com/

Tools:

- Red Hat OpenShift Data Foundation Supportability and Interoperability Checker - https://access.redhat.com/labs/odfsi/
- Red Hat Storage Recovery Calculator - https://access.redhat.com/labs/rhsrc/
- Red Hat ODF Sizing Tool -https://access.redhat.com/labs/ocsst/

Training:

- DO370 - Enterprise Kubernetes Storage with Red Hat OpenShift Data Foundation - https://www.redhat.com/en/services/training/do370-enterprise-kubernetes-storage-with-red-hat-openshift-data-foundation 
- EX370 Red Hat Certified Specialist in OpenShift Data Foundation exam - https://www.redhat.com/en/services/training/ex370-red-hat-certified-specialist-in-openshift-data-foundation-exam

## Prerequisites

The RHODF Operator will install its components only on nodes labeled with `cluster.ocs.openshift.io/openshift-storage=''`.

Nodes can be labeled with:

```console
oc label nodes <NodeName> cluster.ocs.openshift.io/openshift-storage=''
```

>  Note: A minimum of 3 nodes are required.

Nodes can be dedicated to solely RHODF workloads by tainting them with `node.ocs.openshift.io/storage=true:NoSchedule`.

```console
oc adm taint nodes <NodeNames> node.ocs.openshift.io/storage=true:NoSchedule
```

> Note: The dedicated/tainted nodes will only run RHODF components. The nodes will not run any apps. Therefore, if you taint, you need to have additional worker nodes that are untainted. If you don't, you will be unable to run any other apps in your OpenShift cluster.

Also, if deploying dedicated nodes, the nodes should be labeled with `node-role.kubernetes.io/infra: ""` to designate the dedicated RHODF nodes as infra nodes. See [Infrastructure Nodes in OpenShift 4](https://access.redhat.com/solutions/5034771)

```console
oc label node <node-name> node-role.kubernetes.io/infra=
```

## kustomization.yaml

Use to by deploy manifests together

```console
oc apply -k .
```

:warning: Not needed if you are applying the manifest one at a time. This file is helpful when managing a cluster using GitOps.

## namespace-openshift-storage.yaml

Creates namespace `openshift-storage` namespace

```console
oc apply -f namespace-openshift-storage.yaml
```

## operatorgroup-openshift-storage-operatorgroup.yaml

Creates operatorgroup in openshift-storage namespace

```console
oc apply -f operatorgroup-openshift-storage-operatorgroup.yaml
```

## subscription-odf-operator.yaml

Adds RHODF subscription to RHOCP cluster 

```console
oc apply -f subscription-odf-operator.yaml
```

Edit this file with the desired RHODF version.

## storagecluster-ocs-storagecluster.yaml

Creates the storage cluster

```console
oc apply -f storagecluster-ocs-storagecluster
```

Edit this file with the respective size of each disk and the storage class that should be used to provision the OSD disks.

## Tips

Using the Rook-Ceph toolbox to check on the Ceph backing storage:

```console
oc patch OCSInitialization ocsinit -n openshift-storage --type json --patch  '[{ "op": "replace", "path": "/spec/enableCephTools", "value": true }]'

oc rsh -n openshift-storage $(oc get pod -n openshift-storage -l app=rook-ceph-tools -o jsonpath='{.items[0].metadata.name}')
```

