[![Build Status](https://travis-ci.com/onaio/ansible-thumbor.svg?branch=master)](https://travis-ci.com/onaio/ansible-thumbor)

# Thumbor role
This role was forked from [mediapeers/ansible-role-thumbor](https://github.com/mediapeers/ansible-role-thumbor).
Ansible role that installs [Thumbor](https://github.com/thumbor/thumbor) and sets it up for production use.
It uses [supervisord](http://supervisord.org/) to spawn mulitple Thumbor server processes and puts Nginx in front of it, to loadbalance
between them and provide a robust webserver for access from the outside.

Also this role expects to use an S3 bucket as result storage and specific namespace(s) configured as allowed image source.

The normal [image storage](https://github.com/thumbor/thumbor/wiki/Image-storage) (source image caching) is done on the normal filesystem. Make sure to set a expiration time that matches your
scenario to not flood your harddisk. Use the thumbor_storage_expiration variable and point the thumbor_storage_path to a big enough volume.

This role is not only designed to setup Thumbor as an image scaling service but also as an uploading service.
Also unsafe URLs are disabled, meaning you can only use the service with knowing the secret signing key. See `thumbor_signing_key` variable.

## Requirements
This role is built for Ubuntu server 18.04 but also might work on other Debian based distros.
Also you need an AWS S3 bucket setup and the instance this runs on should assume an IAM role (or user credentials in .aws/ or set the AWS credentials env vars via the variables) to make the
[AWS plugin](https://github.com/thumbor-community/aws) work (which uses [Boto](https://boto3.readthedocs.org/en/latest/guide/quickstart.html#configuration) to connect to S3).

## Role Variables
This is the list of role variables with their default values:

* `thumbor_signing_key: ABC123` - Overwrite this to make your thumbor secure! Key that's used to sign requests to Thumbor
* `thumbor_version: 6.7.0` - optional parameter to restrict Thumbor version number more than tc_aws does
* `thumbor_aws_plugin_version: 6.2.12` - Version of the [tc_aws plugin](https://github.com/thumbor-community/aws) for thumbor
* `thumbor_user: ubuntu` - User that runs thumbor (through supervisord)
* `thumbor_config_dir: /etc/thumbor` - Dir that holds the thumbor config files
* `thumbor_log_dir: /var/log/thumbor`
* `thumbor_allowed_sources: []` - Allowed domains used as Thumbor picture input.
* `thumbor_client_side_cache_duration: 24` - client side cache duration in hours
* `thumbor_result_storage_bucket: 'my-namespace-thumbor-cache'` - The bucket name for the result storage
* `thumbor_result_storage_path: result_storage` - The path (bucket folder) where results are cached
* `thumbor_result_storage_expiration: 24` - Result storage cache expiration time in hours
* `thumbor_storage_expiration: 48` - Source image storage cache expiration time in hours
* `thumbor_storage_path: /var/tmp/thumbor/storage` - Location for images storage cache, make sure it's on a volume big enough
* `thumbor_s3_aws_region: us-east-1` - AWS Region for S3 bucket (the aws plugin). If your instance assumes an IAM role you can set this and avoid an boto/aws config file completely
* `thumbor_s3_create_bucket: false` - This will create the bucket on S3 unless set to false. Make sure you have a working AWS/Boto config to grant S3 permissions
* `supervisord_log_dir: /var/log/supervisor` - Log dir for the supervisord service
* `nginx_graylog_server: log.server` - Graylog server for Nginx logs
* `nginx_log_dir: /var/log/nginx` - Nginx log dir
* `nginx_status_page: /nginx_status` - Nginx status page you can use for monitoring

The following variables are not defined by defaulat
* `thumbor_upload_enabled: false` - Enable upload functionality. You want this set to true if you'd like to upload to s3
* `thumbor_upload_photo_storage: <path>` - Upload photo storage path
* `thumbor_upload_put_allowed: false` - Enable image overwriting
* `thumbor_tc_aws_storage_bucket: ""` - S3 storage bucket
* `thumbor_tc_aws_result_storage_bucket: ""` - S3 result storage bucket
* `thumbor_tc_aws_enable_http_loader: false` - Whether to use thumbor aws lib as the http loader. You want to set this to true if you'd like to upload to s3
* `thumbor_storage_module: "tc_aws.storages.s3_storage"` - The thumbor aws python storage module to use
* `thumbor_result_storage_module: "tc_aws.storages.s3_storage"` - The thumbor aws python result storage module to use
* `thumbor_aws_access_key_id : <aws_access_key_id>` - The aws access key id to use. You want to set this if you'd like to upload to s3 and don't have credentials file in ~/.aws
* `thumbor_aws_secret_access_key: <aws_secret_access_key>`  - The aws access key to use. You want to set this if you'd like to upload to s3 and don't have credentials file in ~/.aws
* `thumbor_sentry_dsn_url: <thumbor_sentry_dsn_url>` - Sentry DSN url

There is a config for the Nginx role in `vars/main.yml`. It's set to work with thumbor supervisord setup. But you can throw out stuff you don't
need if you want. Make sure you keep Nginx upstream server config in sync with the Thumbor server processes started by supervisord.

## Dependencies
Depends on the [mediapeers.nginx](https://galaxy.ansible.com/mediapeers/nginx/) Ansible role. Add the Nginx role to your project
with the Ansible Galaxy command (`ansible-galaxy install mediapeers.nginx`) or add directly as Git submodule (repo [here](https://github.com/mediapeers/ansible-role-nginx)).
Make sure it's in `roles/mediapeers.nginx` to make this thumbor role works.

## Example Playbook
This is an example on how to integrate this role into your playbook:
```yaml
- hosts: servers
  vars:
    thumbor_signing_key: 123ABC123Supersecret
    thumbor_allowed_sources:
      - my-s3-namespace-.*s3.amazonaws.com
      - some-domain.com
    thumbor_result_storage_bucket: "my-result-storage-bucket"
    # and other vars you want to override
  roles:
    - mediapeers.thumbor
  tasks:
    # other tasks
```

## License
BSD, as is.

## Author Information
Stefan Horning <horning@mediapeers.com>
