# pgbackup BOSH release
## License
MIT, see LICENSE file.
## Usage: Configuration & Delpoyment

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
bosh upload release releases/pgbackup-3.yml
```

Next edit the manifest file provided with the release (manifest/postgres-backup.yml).

This release support only the Openstack IaaS and the bosh manifest v2.
```sh
cd  manifests
bosh deployment manifests/postgres-backup.yml
```

Add to the list of known `releases: `

```yaml
releases:
- name: pgbackup
  version: latest
```


Add properties such as tags.

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
 - name: pgbackup_TARGETED_DEPLOYMENT_NAME
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

The schedule section use the same syntax as the cron jobs under linux system.
##### how to configure the backup job  : https://help.ubuntu.com/community/CronHowto