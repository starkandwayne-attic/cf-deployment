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

Notes
======================================

For more information, check out the [Genesis][1] repo, or `genesis help`.
You can download the Genesis program from [Github][1]

[1]: https://github.com/starkandwayne/genesis
