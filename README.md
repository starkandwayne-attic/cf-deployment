Cloud Foundry Deployments
======================================

This repository acts as an upstream repository of YAML templates for use
in a Cloud Foundry BOSH deployment, managed via the [Gensis][1] utility.

Creating a new Cloud Foundry Deployment
======================================

To create a new [Genesis][1] based deployment of Cloud Foundry, run
`genesis new deployment --template cf`. This will create a new repo
called `cf-deployments` for you, and pull in the
`github.com/starkandwayne/Cloud Foundry-deployment` repo as the `upstream` remote,
copying the contents of `global/*` into the new `cf-deployments` repo.

This allows you to easily diverge from the upstream templates to suit your
environment, and while also being able to pull in changes from upstream down
the road.

Custom Sites
======================================

To create a new site, run `genesis new site --template <site-type>`. This
will copy in the latest data from the `upstream` remote's `.templates/<site-type>`
directory.


Deploying on BOSH-Lite
======================================

Deploying CF with these templates on BOSH-Lite is *really* easy:

```
$ genesis new deployment --template cf
$ cd cf-deployments
$ genesis new site --template bosh-lite my-site-name # Perhaps 'bosh-lite' or 'macbook' for your site names?
$ genesis new env my-site-name my-environment # Perhaps 'testing', 'playground', or, 'try-anything' for env names?
$ cd my-site-name my-environment
$ make deploy
```

Everything should be all set and deploy without a hitch. When you create a
new environment, `genesis` will prompt you for a Vault to use, and generate
all the credentials/certs you need to get up and running.

We've built these templates to match as closely as possible the experience
one would get when deploying `cf-release` on `bosh-lite`. There were a couple
differences, however:

1. The URL for your CF API is `https://api.system.bosh-lite.com`
   (traditionally it had been `https://api.bosh-lite.com`). We add the
   explicit `system.` to all the system domains for clarity, and to help
   prevent unanticipated outages from apps or route creation that overlaps
   with things like `login`, or `api`.
2. The database is provided from [postgres-boshrelease](https://github.com/cloudfoundry-community/postgres-boshrelease),
   rather than the one built into [cf-release](https://github.com/cloudfoundry/cf-release).
   This was done because that release is currently used in vSphere deployments,
   to provide a database with a hot-standby replica, as there is no RDS-like
   solution available for vSphere, and HA postgres solutions are still being
   evaluated.

Once deployed, you can hit your bosh-lite Cloud Foundry at https://api.system.bosh-lite.com/


Deploying on AWS
======================================

Deploying on vSphere
======================================

Notes
======================================

For more information, check out the [Genesis][1] repo, or `genesis help`.
You can download the Genesis program from [Github][1]

[1]: https://github.com/starkandwayne/genesis
