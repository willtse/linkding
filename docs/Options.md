# Options

This document lists the options that linkding can be configured with and explains how to use them in the individual install scenarios.

## Using options

### Docker

Options are passed as environment variables to the Docker container by using the `-e` argument when using `docker run`. For example:

```
docker run --name linkding -p 9090:9090 -d -e LD_DISABLE_URL_VALIDATION=True sissbruecker/linkding:latest
```

For multiple options, use one `-e` argument per option.

### Docker-compose

For docker-compose options are configured using an `.env` file. 
Follow the docker-compose setup in the README and copy `.env.sample` to `.env`. Then modify the options in `.env`.

### Manual setup

All options need to be defined as environment variables in the environment that linkding runs in.

## List of options

### `LD_DISABLE_BACKGROUND_TASKS`

Values: `True`, `False` | Default = `False`

Disables background tasks, such as creating snapshots for bookmarks on the [the Internet Archive Wayback Machine](https://archive.org/web/).
Enabling this flag will prevent the background task processor from starting up, and prevents scheduling tasks.
This might be useful if you are experiencing performance issues or other problematic behaviour due to background task processing.

### `LD_DISABLE_URL_VALIDATION`

Values: `True`, `False` | Default = `False`

Completely disables URL validation for bookmarks.
This can be useful if you intend to store non fully qualified domain name URLs, such as network paths, or you want to store URLs that use another protocol than `http` or `https`.

### `LD_REQUEST_TIMEOUT`

Values: `Integer` as seconds | Default = `60`

Configures the request timeout in the uwsgi application server. This can be useful if you want to import a bookmark file with a high number of bookmarks and run into request timeouts.

### `LD_SERVER_PORT`

Values: Valid port number | Default = `9090`

Allows to set a custom port for the UWSGI server running in the container. While Docker containers have their own IP address namespace and port collisions are impossible to achieve, there are other container solutions that share one. Podman, for example, runs all containers in a pod under one namespace, which results in every port only being allowed to be assigned once. This option allows to set a custom port in order to avoid collisions with other containers.

### `LD_CONTEXT_PATH`

Values: `String` | Default = None

Allows configuring the context path of the website. Useful for setting up Nginx reverse proxy.
The context path must end with a slash. For example: `linkding/`

### `LD_ENABLE_AUTH_PROXY`

Values: `True`, `False` | Default = `False`

Enables support for authentication proxies such as Authelia.
This effectively disables credentials-based authentication and instead authenticates users if a specific request header contains a known username.
You must make sure that your proxy (nginx, Traefik, Caddy, ...) forwards this header from your auth proxy to linkding. Check the documentation of your auth proxy and your reverse proxy on how to correctly set this up.

Note that this does not automatically create new users, you still need to create users as described in the README, and users need to have the same username as in the auth proxy.

Enabling this setting also requires configuring the following options:
- `LD_AUTH_PROXY_USERNAME_HEADER` - The name of the request header that the auth proxy passes to the proxied application (linkding in this case), so that the application can identify the user.
Check the documentation of your auth proxy to get this information.
Note that the request headers are rewritten in linkding: all HTTP headers are prefixed with `HTTP_`, all letters are in uppercase, and dashes are replaced with underscores.
For example, for Authelia, which passes the `Remote-User` HTTP header, the `LD_AUTH_PROXY_USERNAME_HEADER` needs to be configured as `HTTP_REMOTE_USER`.
- `LD_AUTH_PROXY_LOGOUT_URL` - The URL that linkding should redirect to after a logout.
By default, the logout redirects to the login URL, which means the user will be automatically authenticated again.
Instead, you might want to configure the logout URL of the auth proxy here.
