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

Deploying CF on AWS from these templates should get you a production-worthy CF deployment.
Credentials and certs will be unique and stored in Vault, all services will be HA. The
CCDB/UAADB are expected to use RDS, and the CF blobstore is expected to use S3. Amazon ELBs
will be used in front of the gorouters for load balancing, and SSL termination.

Things you will need to bring to the table:

1. Three subnets (each on a different AZ) for the majority of CF to run on. Due to the
   Raft consensus protocol used in etcd and Consul, three AZs are required to properly
   survive any single AZ failing (there must always be at least `n/2 + 1` nodes up to
   constitute a functioning quorum).
2. Two subnets (each on a different AZ) for the gorouter VMs, with an internet gateway
   attached. This will enable you to isolate the traffic inbound from the internet from
   the rest of the CF deployment.
3. Two RDS instances (with read-replicas to run backups against). One will be for the
   `uaadb`, another for the `ccdb`.
4. An SSL certificate signed for the domain you want to run Cloud Foundry on.
5. An ELB configured to forward HTTP, HTTPS, and TCP+SSL on port 4443. It should be given
   the SSL cert/private key mentioned above. HTTP/HTTPS will handle most traffic going to
   Cloud Foundry. The SSL-encrypted port 4443 will be used for secure websocket connections,
   as AWS ELBs cannot support websockets when in HTTP/HTTPS modes.
6. AWS credentials to create the S3 buckets for your Cloud Foundry blobstore.
7. DNS for your CF domain pointed at your ELB.

Parameters that will need to be filled out:

1. Networking - You'll need to define 3 CF subnets and 2 Router subnets
2. ELBs - You'll need to define the ELB that will be in front of the gorouters.
3. CF Security Group Rules - We add three sets of rules for apps to have access to by 
   default - load_balancer, services, and user_bosh_deployments. The load_balancer group
   should have a rule allowing access to the public IP(s) of the Cloud Foundry installation,
   so that apps are able to talk to other apps. The services group should have rules 
   allowing access to the internal IPs of the services networks
4. Cloud Foundry base domain - we will prepend this with `*.system.` and `*.run.` to create
   the CF system + app default domains.
5. UAADB + CCDB connection info - This should be the RDS hostname/user/password. We default
   to mysql, so if you use postgres on RDS, that will need to be updated as well
6. Blobstore configuration - We expect to be using S3 (via fog) for the CF blobstore. It
   will need keys, and a region defined.

Deploying on vSphere
======================================

Deploying CF on vSphere from these templates should get you most of the way to a production
worthy CF deployment. Credentials and certs will be unique and stored in Vault. All services
will be HA. The CCDB/UAADB will be running on postgres_z1, with an automated replica on
postgres_z2 (manual failover is required at this time). There is no load balancer layer
or SSL termination layer provided, as must people will be bringing their own hardware
solution (F5s, etc) to the table for this.

Prerequisites:

1. A hardware load balancer, equipped with SSL certificates matching your CF domain
2. DNS for your CF domain pointed at the above load balancer.
3. Three subnets (each on a different AZ) for the majority of CF to run on. Due to the
   Raft consensus protocol used in etcd and Consul, three AZs are required to properly
   survive any single AZ failing (there must always be at least `n/2 + 1` nodes up to
   constitute a functioning quorum).
4. Two subnets (each on a different AZ) for the gorouter VMs, with an internet gateway
   attached. This will enable you to isolate the traffic inbound from the internet from
   the rest of the CF deployment.
5. S3 (or s3-like) access for the CF blobstore via Fog.

Parameters that will need to be filled out:

1. Networking - You'll need to define 3 CF subnets and 2 Router subnets
2. CF Security Group Rules - We add three sets of rules for apps to have access to by 
   default - load_balancer, services, and user_bosh_deployments. The load_balancer group
   should have a rule allowing access to the public IP(s) of the Cloud Foundry installation,
   so that apps are able to talk to other apps. The services group should have rules 
   allowing access to the internal IPs of the services networks
3. Cloud Foundry base domain - we will prepend this with `*.system.` and `*.run.` to create
   the CF system + app default domains.
4. Blobstore configuration - We expect to be using S3 (via fog) for the CF blobstore. It
   will need keys, and a region defined.

Notes
======================================

For more information, check out the [Genesis][1] repo, or `genesis help`.
You can download the Genesis program from [Github][1]

[1]: https://github.com/starkandwayne/genesis
