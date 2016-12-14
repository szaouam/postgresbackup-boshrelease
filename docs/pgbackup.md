# pgbackup Project Specific Notes

## Openstack Deployment
If you want to deploy the pgbackup release under  an Openstack environment your need to follow the steps bellow.


Be sure to first target your BOSH Director:
```sh
bosh target $BOSH_HOST
```
Now clone this release and cd into the directory:
```sh
git clone https://.../pgbackup-boshrelease.git
cd pgbackup-boshrelease
```
If you intend on using a final release upload it like so:
```sh
bosh upload release releases/pgbackup-2.yml
```

Next edit the manifest file provided with the release (manifest/postgres-backup.yml).

This release aims to support both v1 and v2 manifest. Currently, only Openstack templates are offered by default, 
support for other Iaas are welcome.

The first step is to choose under wish bosh you want to use de the postgres-backup release :
```sh
bosh deployment your-bosh-deployment.yml
```

Add to the list of known `releases` :

```yaml
releases:
- name: pgbackup
  version: latest
```

Add the pgbackup property.

```yaml
properties:
# ... 
  pgbackup:
```

Now, for every `instances: ` entry you wish to collocate this release with under `jobs:` add the following in the `templates: ` section:

```yaml
  - name: pgbackup
    release: pgbackup
```

Now you can deploy,

```yaml
bosh -n deploy
```

You can run backup for multiple deployments by replicating the section bellow (from the manifests/postgres-backup.yml) :

```yaml
 - name: pgbackup_DEPLOYMENT_NAME
   networks:
   - name: REPLACE_WITH_YOUR_NETWORK_NAME
     static_ips:
     - 10.250.2.45 # list of ips for pgbackup instances
     cloud_properties:
       security_groups:
       - tf-bosh
       - default
   vm_type: default
   stemcell: trusty
   azs: [z1]
   instances: 1
   jobs:
     - name: pgbackup
       release: pgbackup
   properties:
     pgbackup:
       host: REPLACE_WITH_TARGETED_DEPLOYMENT_IP_ADRESS
       port: REPLACE_WITH_TARGETED_DEPLOYMENT_POSTGRES_PORT
       username: REPLACE_WITH_TARGETED_DEPLOYMENT_USERNAME
       password: REPLACE_WITH_TARGETED_DEPLOYMENT_PASSWORD
       schedule: 
         minute: '*'
         hour: '*'
         monthday: '*'
         month: '*'
         weekday: '*'
       s3:
         bucket: REPLACE_WITH_YOUR_BUCKET_NAME
         access_key_id: REPLACE_WITH_ACCESS_KEY_ID
         secret_access_key: REPLACE_WITH_SECRET_ACCESS_KEY
         endpoint: REPLACE_WITH_ENDPOINT_WITHOUT_HTTP_OR_HTTPS

```

The schedule property use the same syntax as the cron jobs under linux system.
##### how to configure the backup job  : https://help.ubuntu.com/community/CronHowto