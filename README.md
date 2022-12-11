# laurivan.outline

This role installs Outline via Docker.

## Requirements

None

## Role Variables

All variables are listed below (see also `defaults/main.yml`).

### Outline Core Variables

Outline requires a couple of secrets for data encryption:

```yml
outline_secret_key: 'changeme'
outline_utils_secret: 'changeme'
```

You also need to specify the deployment type. Usually it's `production`

```yml
outline_deployment: ''
```

You also need to define how you access outline:

- `outline_port` is the port mapping in Docker. Outline runs at port 3000, which is alos the default
- `outline_url` is the public URL where we see Outline. If you use reverse proxy mapping, put the URL of the reverse proxy (in my case *[this one](https://wiki.home.laurivan.com)*).
- `outline_force_https` will run with HTTPS if true. you can define it as *false* If you're behind a proxy or you don't have a certificate. It defaults to `false`.
- `outline_enable_updates` will enable updates if true. Please read [the documentation](https://app.getoutline.com/s/770a97da-13e5-401e-9f8a-37949c19f97e/) for what this implies (e.g. telemetry)
- Define `outline_cdn_url` if you have a CDN. Defaults to *empty*

**Note**: `outline_url` will define the authentication redirect url for e.g. authentik

You can define which debug messages to be logged via `outline_debug`.

### Storage

Following values are defined for the docker-compose:

```yml
outline_volume_base: "/mnt/outline"
outline_setup_path: '{{ outline_volume_base }}/config'
outline_volume_redis: "{{ outline_volume_base }}/redis"
outline_volume_db: "{{ outline_volume_base }}/db"
outline_volume_s3: "{{ outline_volume_base }}/s3"
```

Please note that `outline_volume_db` and `outline_volume_s3` are actually created only if local posstgres and fake_s3 containers are created by configuration below.

You can specify a logo too via `outline_team_logo_url`. By default this is empty.

You can also change the default language via `outline_language`. The role defaults the language to *en_US*.

### Authentication

Outline authentication can happen via:

- OIDC
- Google authentication
- Slack

You need to define at least one of them.

#### OIDC

OIDC parameters are

```yml
oidc_client_id:
oidc_client_secret:
oidc_auth_uri:
oidc_token_uri:
oidc_userinfo_uri:
```

Your authentication app should provide you all the above. I use something along the lines:

```yml
oidc_client_id: "changeme"
oidc_client_secret: "changeme"
oidc_auth_uri: "https://sso.laurivan.com/application/o/authorize/"
oidc_token_uri: "https://sso.laurivan.com/application/o/token/"
oidc_userinfo_uri: "https://sso.laurivan.com/application/o/userinfo/"
oidc_username_claim: "preferred_username"
```

**Note**: you will probably need to provide the redirect URL to the authentication application. For Authentik, you can find it in the **Provider** for the specific application.

#### Google ID

You need to define:

```yml
outline_google_client_id:
outline_google_client_secret:
```

#### Slack

You need to define

```yml
outline_slack_client_id:
outline_slack_client_secret:
```

### Database

You need to assign a database to Outline. This role allows you to launch Postgres in a container via:

```yml
outline_db_schema: "postgres"
outline_db_host: "postgres"
outline_db_port: "5432"
outline_db_user: "postgres"
outline_db_password: "changeme"
outline_db: "outline"
```

If the db_host is not "postgres", then we assume the db is external and not spin up the docker container.

By default, PostgreSQL is not secured. If you have a secure database instance, set the `outline_db_ssl` variable to "enable".

### S3

We define the following variables:

```yml
outline_fake_s3: true
outline_fake_s3_port: 4569
outline_aws_access_key_id:
outline_aws_secret_access_key:
outline_aws_region:
outline_aws_s3_upload_bucket_url: "http://s3:4569"
outline_aws_s3_upload_bucket_name: outline-bucket
outline_aws_s3_upload_max_size: "26214400"
outline_aws_s3_force_path_style: "true"
outline_aws_s3_acl: "private"
```

You need S3 (or S3-like) storage for e.g. uploaded files. By default, the role spins up the fake S3 only if `fake_s3` variable is true.

I use [MinIO](https://min.io/) with something like:

```yml
outline_fake_s3: ""
outline_aws_access_key_id: "change me"
outline_aws_secret_access_key: "change me"
outline_aws_region: "my-rack"
outline_aws_s3_upload_bucket_url: "http://minio,example.com:9000"
outline_aws_s3_upload_max_size: "26214400"
outline_aws_s3_force_path_style: "true"
outline_aws_s3_acl: "private"
```

### Email

Outline can send notification emails if you set up the SMTP variables:

```yml
outline_smtp_host:
outline_smtp_port:
outline_smtp_username:
outline_smtp_password:
outline_smtp_from_email:
outline_smtp_reply_email:
```

## Dependencies

You need a machine with docker and docker-compose installed.

## Example Playbook

```yml
- hosts: servers
  roles:
      - 'laurivan.outline'
```

## License

MIT

## Author Information

This role was created in 2022 by [Laur Ivan](https://www.laurivan.com).
