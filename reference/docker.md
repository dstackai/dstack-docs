# Docker

Here's how to run `dstack` in a Docker container:

```bash
docker run -it --rm --name dstack \
    -p 8080:8080 \
    dstackai/dstack
```

Be aware that all the data will be lost once the Docker container gets removed. To persist the data mount the `~/.dstack` folder:

```bash
docker run -it --rm --name dstack \
    -p 8080:8080 \
    -v ~/.dstack:/.dstack \
    dstackai/dstack
```

Here's the list of all environment variables that are supported by `dstack`:

| Environment variable | Description | Default value |
| :--- | :--- | :--- |
| `DSTACK_INTERNAL_PORT` | The port`dstack`is running on inside the container. | `8080` |
| `DSTACK_PORT` | The external port of the host if it's different from the internal port. | `8080` |
| `DSTACK_HOST_NAME` | The external name of the host if it's different from `localhost`. | `localhost` |
| `DSTACK_SSL` | `true` if the hostname is available via SSL. | `false` |
| `DSTACK_USER` | Not required. The default name of the admin user. | `dstack` |
| `DSTACK_PASSWORD` | The password of the admin user. Not required. If not set, it's generated randomly on the first start. |  |
| `DSTACK_SMTP_HOST` | The hostname of the SMPT server for user authorization and notification emails. Not required. |  |
| `DSTACK_SMTP_PORT` | The port of the SMPT server for user authorization and notification emails. Not required. |  |
| `DSTACK_SMTP_USER` | The user of the SMPT server for user authorization and notification emails. Not required. |  |
| `DSTACK_SMTP_PASSWORD` | The password of the SMPT server for user authorization and notification emails. Not required. |  |
| `DSTACK_SMTP_STARTTLS` | `true` if the SMPT server for user authorization and notification emails is using TLS. | `true` |
| `DSTACK_SMTP_FROM` | The email authorization and notifications emails are sent from. Required if SMTP is used. |  |
| `DSTACK_ADMIN_EMAIL` | The email of the administrator of the `dstack` server. Required if SMTP is used. |  |

