Kubernetes Dashboard
====================

A web dashboard from the Kubernetes team.

[Quick instructions](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).

[On GitHub](https://github.com/kubernetes/dashboard).

We are creating an `admin` serviceaccount for it with cluster-admin privileges.

Use [`admin-token`](admin-token) to get its token.

Unfortunately, it works for a while and then we get CrashLoopBackOff errors on the pod. :()
