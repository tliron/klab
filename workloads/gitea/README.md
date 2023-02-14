Gitea
=====

[Gitea](https://gitea.io/en-us/) hosts git repositories. It's like your own little GitHub!

[On GitHub](https://github.com/go-gitea/gitea).

We are deploying it to the `gitea` namespace using the
[published Helm chart](https://docs.gitea.io/en-us/install-on-kubernetes/).

The exposed base git URL is `http://$(hostname):32100`.

The internal base git URL is `http://gitea-http.gitea:3000`

The default admin user is `administrator`, password `administrator`.

For testing you can use git in the Gitea main pod via [`shell`](shell).

Admin operations on Gitea can be handled in two different ways:

1) Using the [`admin/gitea`](admin/gitea) CLI, or
2) Using [`admin/api`](admin/api) to call the [Swagger endpoints](https://try.gitea.io/api/swagger).
   Most of these API calls require token authentication, which we store in the `admin/tokens`
   directory and generate using [`admin/access-token`](admin/access-token).
