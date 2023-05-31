# Docker Autoheal

Monitor and restart unhealthy docker containers.
Monitor and restart unhealthy docker containers.
This functionality was proposed to be included with the addition of `HEALTHCHECK`, however didn't make the cut.
This container is a stand-in till there is native support for `--exit-on-unhealthy` https://github.com/docker/docker/pull/22719 and https://github.com/moby/moby/issues/28400.

## Supported tags and Dockerfile links
- `latest` [(*Dockerfile*)](https://github.com/modem7/docker-autoheal/blob/master/Dockerfile)
- `pinned`

![](https://img.shields.io/docker/pulls/modem7/docker-autoheal "Total docker pulls") [![](https://images.microbadger.com/badges/image/modem7/docker-autoheal.svg)](http://microbadger.com/images/modem7/docker-autoheal "Docker layer breakdown")

## How to use
### UNIX socket passthrough
```bash
docker run -d \
    --name autoheal \
    --restart=always \
    -e AUTOHEAL_CONTAINER_LABEL=all \
    -v /var/run/docker.sock:/var/run/docker.sock \
    modem7/docker-autoheal
```
### TCP socket
```bash
docker run -d \
    --name autoheal \
    --restart=always \
    -e AUTOHEAL_CONTAINER_LABEL=all \
    -e DOCKER_SOCK=tcps://HOST:PORT \
    -v /path/to/certs/:/certs/:ro \
    modem7/docker-autoheal
```
a) Apply the label `autoheal=true` to your container to have it watched.

b) Set ENV `AUTOHEAL_CONTAINER_LABEL=all` to watch all running containers.

c) Set ENV `AUTOHEAL_CONTAINER_LABEL` to existing label name that has the value `true`.

Note: You can also exclude a container (eg. if using `AUTOHEAL_CONTAINER_LABEL=all`) by setting the label `autoheal=false`.

Note: You must apply `HEALTHCHECK` to your docker images first. See https://docs.docker.com/engine/reference/builder/#healthcheck for details.
Use `tcp://` for unencrypted tcp and `tcps://` for TLS enabled connection.
See https://docs.docker.com/engine/security/https/ for how to configure TCP with mTLS.

The certificates, and keys need these names:
* ca.pem
* client-cert.pem
* client-key.pem

### Change Timezone
If you need the timezone to match the local machine, you can map the `/etc/localtime` into the container.
```
docker run ... -v /etc/localtime:/etc/localtime:ro
```

### Apprise configuration
a) Start the Apprise docker container. See: https://hub.docker.com/r/caronc/apprise

b) Create an Apprise configuration for Autoheal to use:
http://localhost:8000/cfg/autoheal

See the Apprise wiki for configuration details: https://github.com/caronc/apprise/wiki

c) Set the `APPRISE_URL` environment variable to use the Apprise configuration:
`APPRISE_URL="http://localhost:8000/notify/autoheal"`

## ENV Defaults
```
AUTOHEAL_CONTAINER_LABEL=autoheal
AUTOHEAL_INTERVAL=5   # check every 5 seconds
AUTOHEAL_START_PERIOD=0   # wait 0 seconds before first health check
AUTOHEAL_DEFAULT_STOP_TIMEOUT=10   # Docker waits max 10 seconds (the Docker default) for a container to stop before killing during restarts (container overridable via label, see below)
DOCKER_SOCK=/var/run/docker.sock   # Unix socket for curl requests to Docker API
CURL_TIMEOUT=30     # --max-time seconds for curl requests to Docker API
WEBHOOK_URL=""    # post message to the webhook if a container was restarted (or restart failed)
WEBHOOK_JSON_KEY="content"    # the key to use for the message in the json body of the request to the webhook url
APPRISE_URL=""    # post message to Apprise if a container was restarted (or restart failed)
POST_RESTART_SCRIPT=""    # Run the specified script if a container was restarted (or restart failed). Script is run from inside the container. Make sure to mount a host directory with the script you want to run. e.g. `-v ./scripts:/scripts`
```

### Optional Container Labels
```
autoheal.stop.timeout=20        # Per containers override for stop timeout seconds during restart
```

### Post-Restart Script

Here's an example of how you can execute a script after a restart.

The following values are passed as arguments: `CONTAINER_NAME`, `CONTAINER_SHORT_ID`, `CONTAINER_STATE`, `RESTART_TIMEOUT`.

```bash
docker build -t autoheal .

docker run -d \
    -e AUTOHEAL_CONTAINER_LABEL=all \
    -e POST_RESTART_SCRIPT=/scripts/post_restart.sh \
    -v ./scripts:/scripts \
    -v /var/run/docker.sock:/var/run/docker.sock \
    autoheal
```

## Testing
```bash
docker build -t autoheal .

docker run -d \
    -e AUTOHEAL_CONTAINER_LABEL=all \
    -e POST_RESTART_SCRIPT=/scripts/post_restart.sh
    -v /var/run/docker.sock:/var/run/docker.sock \
    autoheal
```
