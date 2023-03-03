# RHODF

# Table of Contents 

- [Prerequisites](#Prerequisites)
- [kustomization.yaml](#kustomization.yml)
- [namespace-openshift-storage.yaml](#namespace-openshift-storage.yaml)
- [operatorgroup-openshift-storage-operatorgroup.yaml](#operatorgroup-openshift-storage-operatorgroup.yaml)
- [subscription-odf-operator.yaml](#subscription-odf-operator.yaml)
- [storagecluster-ocs-storagecluster.yaml](storagecluster-ocs-storagecluster.yaml)

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
