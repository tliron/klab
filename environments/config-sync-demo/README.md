Config Sync Demo
================

Demonstrates [Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview).
We'll show how a cluster's resources can be created and kept in sync with a git repository.

[`start`](start) sets up two KIND clusters: "management" and "edge1".

The management cluster is used just to host Gitea for us. We set up a git repository called 
"config-sync-demo".

On the edge1 cluster we deploy Config Sync.

Run [`deploy`](deploy) to fill the git repository with [this content](assets/deployment-repository/)
and apply [this `RootSync` resource](assets/root-sync.yaml) to the edge1
cluster. It points to our git repository's [`/root/`](assets/deployment-repository/root/)
directory. Config Sync will reconcile it thus:

1) Creates the "network-function" namespace.
2) Adds a `RepoSync` resource pointing to our git repository's
   [`/namespaces/network-function/`](assets/deployment-repository/namespaces/network-function/)
   directory.
3) Gives Config Sync permissions to manage that namespace.

The `RepoSync` will cause Config Sync to deploy a new controller to sync our directory.
After a few minutes you should see the synced resources in the newly created namespace:

    platforms/kind/use edge1
    kubectl get pods --namespace=network-function
