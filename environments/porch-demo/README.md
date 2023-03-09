Porch Demo
==========

Demonstrates [Porch](https://kpt.dev/guides/porch-user-guide) working with
[Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview).
We'll show how kpt packages can be 1) stored and managed in git repositories and
2) deployed to clusters by having them sync with git repositories.

[`start`](start) sets up two KIND clusters: "management" and "edge1".

The management cluster hosts Gitea and Porch. We create
[three Porch `Repository` resources](assets/porch-repositories.yaml) there:

* "egde1" for an internal git repository on our Gitea, also called "edge1".
  We initialize it with [this content](assets/deployment-repository/) to ready it for Config Sync.
  See the [Config Sync demo](../config-sync-demo/) for more detail.
* "blueprints" for an internal git repository on our Gitea, also called "blueprints".
  We initialize it with [this content](assets/blueprints-repository/) because we need at least
  one commit for Porch to be able to use it.
* "external-blueprints" for an external git repository at
  `https://github.com/nephio-project/free5gc-packages.git`.

To see the repositories using `kubectl` or `kpt`:

    kubectl get repository --namespace=porch-demo
    kpt alpha repo get --namespace=porch-demo

Porch will poll these repositories and create a `PackageRevision` with a matching
`PackageRevisionResources` for each revision of each kpt package it finds. (Note
that `PackageRevisionResources` is *not* a custom resource, but rather implemented
by Porch via [API aggregation](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/).)
To list them all using `kubectl` or `kpt`:

    kubectl get packagerevision --namespace=porch-demo
    kpt alpha rpkg get --namespace=porch-demo

To see the contents of `PackageRevisionResources` for an individual kpt package, e.g. an
external blueprint, using `kubectl` or `kpt`:

    BLUEPRINT=$(workloads/porch/package-revision-name porch-demo external-blueprints free5gc-operator)
    kubectl get packagerevisionresources "$BLUEPRINT" --namespace=porch-demo --output=jsonpath="{range .spec.resources.*}{'---\n'}{@}{end}"
    kpt alpha rpkg pull "$BLUEPRINT" --namespace=porch-demo

Create the Blueprint
--------------------

Run [`create-blueprint`](create-blueprint) to create a kpt package in our "blueprints"
repository based on [this content](assets/blueprints/network-function/), which we initialized
with `kpt pkg init` and then modified. The lifecycle is:

### Create the draft

1) We start with a new `PackageRevision` resource for the kpt package. Its "lifecycle" field is "Draft".
   Its "workspaceName" is "v1", but it does not have a "revision" yet.
2) Porch syncs this resource into the "blueprints" git repository by creating a new branch called
   "drafts/network-function/v1" with the contents of the kpt package in a directory named
   "network-function" (the "packageName" field of our `PackageRevision`).

### Propose the draft

1) Our `PackageRevision` resource's "lifecycle" field has changed to "Proposed".
2) Our git repository's branch has been renamed to "proposed/network-function/v1".

### Approve the draft

1) Our `PackageRevision` resource's "lifecycle" field has changed to "Published"
   and its "revision" field set to "v1".
2) Our git repository's branch has been merged into "main" and deleted.
3) The "main" branch HEAD has been tagged as "network-function/v1".

Deploy the Blueprint
--------------------

Now run [`deploy-blueprint-manually`](deploy-blueprint-manually). It will:

1) Pull the contents of kpt package above into a temporary directory. (Of course we already have it
   locally from where we created it above, but for demonstration purposes we'll assume that it was
   created by someone else at an earlier time.)
2) Mutate it by calling the [set-namespace kpt function](https://catalog.kpt.dev/set-namespace/v0.4/)
   to ready it to be deployed. (Note that Config Sync is unaware of the Kptfile, thus we cannot
   rely on declarative kpt mutation for deployment in this combination.)
3) Create a new kpt package for it in our "deployment" repository, following the same lifecycle
   as above.
4) Deploy a [`RootSync`](assets/root-sync.yaml) to the edge1 cluster.

Config Sync will take it from here and after a few minutes you should see the synced resources in
the newly created namespace:

    platforms/kind/use edge1
    kubectl get pods --namespace=network-function-a

Deploy the Blueprint As a Variant
---------------------------------

Now run [`deploy-blueprint-variant`](deploy-blueprint-variant). It will apply
[this `PackageVariant`](assets/variant.yaml).

The Package Variant Controller will then pull our blueprint package from the upstream repository
and from it create a draft `PackageRevision` in the downstream repository.

We have to wait for that `PackageRevision` to be created, and then we follow the same workflow as
above and get the network function deployed in another namespace:

    platforms/kind/use edge1
    kubectl get pods --namespace=network-function-b
