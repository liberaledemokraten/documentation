---
title: 'Add Services'
taxonomy:
    category:
        - vesta
    tag:
        - vesta
        - systemd
---

## Systemd
Within the server, many applications will run as services. Those can be started and stopped using `systemctl`. You may want to add some new services that you can manage that way. You can achieve it by doing as follows:

```sh
/bin/systemctl daemon-reload
/bin/systemctl enable {appname}.service
```

The second line may not work with all applications, so you may have to write a file named `{appname}.service` under `/etc/systemd/system/`. The file may look as follows:

```systemd
[Unit]
Description={shortDescription}

[Service]
Type=simple
User={userToRunTheApp}
Group={groupToRunTheApp}
ExecStart={pathToBinary}
Restart=on-failure
RestartSec=3
StartLimitBurst=3
StartLimitInterval=60
WorkingDirectory=/

[Install]
WantedBy=multi-user.target
```

More information on [ShellHacks](https://www.shellhacks.com/systemd-service-file-example/) and the systemd manual.

## Vesta
! Be warned that the changes to Vesta's binaries may be overriden with an update to Vesta.

!!! When uninstalling the application, don't forget to remove it from vesta as well.

After having added a service to `systemd`, it is also possible to add it to the Vesta console and monitor as well as manage the service from its web UI. To achieve that, follow these steps:, assuming we are adding `example` as a service:

1. add the service to `systemd`
2. Add `EXAMPLE_SYSTEM='example'` as a line in `/usr/local/vesta/config/vesta.conf`
3. Add the following in `/usr/local/vesta/bin/v-list-sys-services`under "Action"

```sh
## Checking EXAMPLE system
if [ ! -z "$EXAMPLE_SYSTEM" ] && [ "$EXAMPLE_SYSTEM" != 'remote' ]; then
    get_srv_state $EXAMPLE_SYSTEM
    data="$data\nNAME='$EXAMPLE_SYSTEM' SYSTEM='example application' STATE='$state'"
    data="$data CPU='$cpu' MEM='$mem' RTIME='$rtime'"
fi
```

4. if the correct status is not displayed in Vesta, you may also add the following in the same file under "searching related pids"

```sh
if [ "$name" = 'example' ]; then
    pids=$(systemctl show example --property=MainPID | sed 's/MainPID=//')
    status=$(systemctl show example --property=SubState | sed 's/SubState=//')
    if [ "$status" = 'dead' ]; then
        state='stopped'
    fi
fi
```

5. To be able to modify the service's configuration through vesta, you should add the following in `/usr/local/vesta/bin/v-change-sys-service-config` under "Defining dst config path"

```sh
case $service in
    example)    dst='{pathToConfigFile}'
esac
```