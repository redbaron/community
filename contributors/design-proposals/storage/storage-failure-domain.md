# Storage specific failure domains

## Goal

Introduce Storage specific failure domains concept.

## Motivation

In Datacenter setup failure domains for storage might not necessary be the same as failure domain for compute, which differs from clouds where they match. There is currently no way to reflect these setups in Kube, which leads to impossible schedulling with stuck pods.

### Impossible scheduling
Let *A*,*B*,*C*,*D* be compute failure domains and *X*, *Y* be storage failure domains. For most cloud providers they are mapped 1:1, but that is not generally the case for Datacenter setups. 


Consider following arrangements where storage failure domains are wider than compute domains:
```
[  A  ][  B  ][  C  ][  D  ]   : compute failure domains
<      X     ><      Y     >   : storage failure domains
```
This can happen when for instance there are multiple SAN storages available to subset of the nodes each or when some nodes have no connectivity to SAN and therefore belong to nonexistent storage failure domain, or if storage provider has builtin concept of failure domains (i.e. Cinder).

In current K8S version 1.9.0 there is no way to express this setup.  


### New failure domains labels
Storage failure domain is by definition depends on a storage system which provides service. There can be multiple of them with different failure domains each. Closest related Kubernetes resource is StorageClass, therefore there should be storage failure domain label per storage class

Proposed label format is: `storage-failure-domain.alpha.kubernetes.io/<storage-class-name>=<domain-name>`

StorageClass: useFailureDomains: true

VolumeProvisioner: label PV with failure domain if StorageClass.useFailureDomain == true

Scheduler predicate:
  1. construct expected label from pvc.storageclass.name
  1. get expected label value from pvInfo(pvc.pvname), if set: filter all nodes with same value; if not set fallback to comparing compue failure domain labels
