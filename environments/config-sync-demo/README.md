Config Sync Demo
================

Sets up two KIND clusters: "management" and "edge1".

The management cluster hosts Gitea for us. We push [this content](assets/repository/) to a git
repository called "config-sync-demo". The structure of this content is based on
[this example](https://github.com/GoogleCloudPlatform/anthos-config-management-samples/tree/main/config-sync-quickstart/multirepo).

On the edge1 cluster we will deploy Config Sync and use [this RootSync](assets/root-sync.yaml) to
to start syncing our git repository's [`/root/`](assets/repository/root/) directory. That will in turn
cause us to [sync this](assets/repository/root/network-function.yaml), which will 1) create a namespace,
2) add a RepoSync resource to it, which will sync the resources in our repository's
[`/namespaces/network-function/`](assets/repository/namespaces/network-function/) directory, and 3) give
Config Sync permissions to manage the namespace.

After a few minutes you should see the synced resources in the newly created namespace:

    platforms/kind/use edge1
    kubectl get pods --namespace=network-function
