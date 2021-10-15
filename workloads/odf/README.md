OpenShift Data Foundation (ODF)
===============================

OpenShift Container Storage
---------------------------

A high-level operator on top of [Rook](https://rook.io/) ([Ceph](https://ceph.io/en/) operator) and
[NooBaa](https://www.noobaa.io/).

* [guide](https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.8/html/deploying_openshift_container_storage_using_bare_metal_infrastructure/deploy-using-local-storage-devices-bm#creating-openshift-container-storage-cluster-on-bare-metal_rhocs)
* [guide](https://red-hat-storage.github.io/ocs-training/training/ocs4/ocs4-install-no-ui.html)
* [code](https://github.com/rook/rook)

*Not* preinstalled in SNO

Installed in namespace: `openshift-storage` (this will also be the name of the cluster)

OCS currently does not support SNO. Thus, we will use Rook directly in order to create our Ceph cluster.

A Ceph cluster comprises the following component types:

* *mon (monitor)*: These are the high-level Ceph cluster itself. Each instance manages a database about the cluster stored
  by default at `/var/lib/rook` on its host. Note that if you run multiple Ceph clusters then you must set each to use a
  different directory. A quorum of at least 3 mon instances is required per cluster. The default for multi-node clusters it to
  place them on separate nodes, so for SNO we need to explicitly enable them to run on the same node.
* *mgr (manager)*: These provide user access to the Ceph cluster. This includes responding to the `ceph` command line, serving
  the web dashboard, and providing Prometheus telemetry. They can be configured with various optional modules. The Ceph cluster
  can scale these out as need. For SNO a single instance will be automatically deployed.
* *osd (object storage daemon)*: These handle low-level local disk I/O, which they expose to clients over the network.
  They are deployed on-demand for every node that is adding its local disk to the cluster. By default multiple instances
  are deployed per node for redunancy but for SNO this is unnecessary, so we configure it to use just one.
* *mds (metadata server daemon)*: These handle a high-level distributed filesystem on top of the raw storage exposed by the osd
  instances. Internally this is achieved by creating a separate storage block for filesystem metadata. They are only deployed if
  a filesystem is explicitly created for the cluster. Note that since Ceph 16 it is possible to create multiple filesystems per
  cluster. Rook always creates double the number of mds instances we request for redundancy. For SNO it is enough to request
  one, thus we will have two running mds instances.
* *crashcollector*: These are deployed per node to collect crash information from other components. For SNO this means that
  only one will be deployed.

Rook notes:

* Rook makes heavy use of Kubernetes finalizers to manage cleaning up of resources such as CephCluster and CephFilesystem.
  Unfortunately they are quite fragile and may hang if the cluster has failed to start properly. To force their deletion you may
  have to manually remove edit the resource and remove the finalizer. You might also have to delete the current rook-ceph-operator
  pod (when a new pod is created it will attempt to reconcile existing Ceph clusters).
* If you delete the cluster then Rook will intentionally _not_ wipe the `/var/lib/rook` (or other mon directory) from the hosts.
  Thus if you want to recreate the cluster you have to manually wipe it from each host in which a mon was running. If you don't
  then you will get various uninformative errors when trying to set up the mon quorum.

https://www.rusinov.ie/en/posts/2020/setting-up-single-node-ceph-cluster-for-kubernetes/


Local Storage Operator
----------------------

* [code](https://github.com/openshift/local-storage-operator)
* [guide](https://docs.openshift.com/container-platform/4.8/storage/persistent_storage/persistent-storage-local.html)

Preinstalled in SNO in namespace: `openshift-local-storage`

You want to also enable "Local Volume Discovery" (this will fill in the "Disks" tab for Nodes)

Precreated StorageClass: `localblock-sc`

Precreated LocalVolumeSet: `local-disks`

    kubectl get localvolumeset local-disks -n openshift-local-storage -o yaml

The operator will create a PersistentVolume named `local-pv-???` for each disk
