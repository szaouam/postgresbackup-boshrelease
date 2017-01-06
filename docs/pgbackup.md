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

You'll need to edit the manifest of the deployment to amend. You may choose to download the manifest from bosh director, or modify a version you saved into you version control. 

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

##Multiple database backup

This release can be used to backup multiple database. If you have n databases to backup, you can create n instances, each one of them will backup one specified database from a specified host.

To use this feature, you need to deploy the postgresbackup-boshrelease using the bosh manifest v2(see. templates/postgres-backup-v2.yml ).


````yaml
name: REPLACE_WITH_DEPLOYMENT_NAME
director_uuid: REPLACE_WITH_DIRECTOR_UID

instance_groups:
- name: pgbackup_first # First instance used to backup the first database
  networks:
  - name: REPLACE_WITH_YOUR_NETWORK_NAME 
    static_ips:
    - 10.10.0.1
    cloud_properties:
      security_groups:
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
      host: REPLACE_WITH_TARGETED_DEPLOYMENT_IP # used for restore
      port: REPLACE_WITH_DATABASE_PORT # used for restore
      username: REPLACE_WITH_YOUR_DATABASE_USERNAME #used for restore
      password: REPLACE_WITH_YOUR_DATABASE_PASSWORD # used for restore
      schedule: #used to schedule backup
        minute: '00'
        hour: '01'
        monthday: '*'
        month: '*'
        weekday: '*'
      s3:
        bucket: REPLACE_WITH_YOUR_BUCKET_NAME
        access_key_id: REPLACE_WITH_YOUR_ACCESS_KEY_ID
        secret_access_key: REPLACE_WITH_YOUR_SECRET_ACCESS_KEY
        endpoint: REPLACE_WITH_YOUR_ENDPOINT_URL # see Notes section for more details
      pgdump:
        host: REPLACE_WITH_TARGETED_DEPLOYMENT_IP
        port: REPLACE_WITH_DATABASE_PORT
        username: REPLACE_WITH_YOUR_DATABASE_USERNAME
        password: REPLACE_WITH_YOUR_DATABASE_PASSWORD
        database: REPLACE_WITH_YOUR_DATABASE_NAME

- name: pgbackup_second # Second instance used to backup the second database
  networks:
  - name: REPLACE_WITH_YOUR_NETWORK_NAME 
    static_ips:
    - 10.10.0.2
    cloud_properties:
      security_groups:
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
      host: REPLACE_WITH_TARGETED_DEPLOYMENT_IP # used for restore
      port: REPLACE_WITH_DATABASE_PORT # used for restore
      username: REPLACE_WITH_YOUR_DATABASE_USERNAME #used for restore
      password: REPLACE_WITH_YOUR_DATABASE_PASSWORD # used for restore
      schedule: #used to schedule backup
        minute: '00'
        hour: '01'
        monthday: '*'
        month: '*'
        weekday: '*'
      s3:
        bucket: REPLACE_WITH_YOUR_BUCKET_NAME
        access_key_id: REPLACE_WITH_YOUR_ACCESS_KEY_ID
        secret_access_key: REPLACE_WITH_YOUR_SECRET_ACCESS_KEY
        endpoint: REPLACE_WITH_YOUR_ENDPOINT_URL # see Notes section for more details
      pgdump:
        host: REPLACE_WITH_TARGETED_DEPLOYMENT_IP
        port: REPLACE_WITH_DATABASE_PORT
        username: REPLACE_WITH_YOUR_DATABASE_USERNAME
        password: REPLACE_WITH_YOUR_DATABASE_PASSWORD
        database: REPLACE_WITH_YOUR_DATABASE_NAME


update:
  canaries: 1
  canary_watch_time: 1000-60000
  max_in_flight: 1
  serial: false
  update_watch_time: 1000-60000

releases:
- name: pgbackup
  version: latest

stemcells:
- alias: trusty
  os: ubuntu-trusty
  version: latest


````

#### Notes

This section will give some useful information that can help you to deploy the postgresbackup-boshrelease

The schedule section is used to set how we want to set the backup. The example bellow mean that we will run the backup process every day at 23h at clock.

````yaml
schedule: #used to schedule backup
    minute: '00'
    hour: '23'
    monthday: '*'
    month: '*'
    weekday: '*'
````

Scheduling backup is base on the cron jobs. See more  : https://en.wikipedia.org/wiki/Cron

The S3 section aim to provide a remote storage, any storage service that implement the S3 protocol.

To set correctly the S3 section of the deployment manifest, you need to follow the recommendations bellows.

* If the link used to access to your bucket is like ``https://storage.mywebsite.com\my-bucket`` or ``http://my-bucket.s3.amazonaws.com`` (sometimes like ``http://my-bucket.s3-aws-region.amazonaws.com`` ), 
the bucket field will be ``bucket: my-bucket``
* The ``access_key_id`` and ``secret_access_key`` are the same that are provided by your s3 storage service.
* If your s3 endpoint is like ``https://storage.mywebsite.com``, the endpoint field will be ``endpoint: storage.mywesbite.com``












