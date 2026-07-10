
# place this example code at /etc/logrotate.d/traefik on your docker host server
# please adjust the custom file path below, where your traefik logs are stored
# please adjust the below traefik container name to send the USR1 signal for log rotation

compress
/path/to/traefik/logs/*.log {
    daily
    maxsize 50M
    rotate 14

    missingok
    notifempty

    compress
    delaycompress

    sharedscripts

    postrotate
        /usr/bin/docker kill --signal="USR1" traefik >/dev/null 2>&1 || true
    endscript
}
