# 2 nodes setup
[ref](https://devopscube.com/setup-consul-cluster-guide/)

# files/dir
unzip consul and put under `/usr/local/bin`

```
mkdir -p /etc/consul.d/scripts
mkdir /var/consul
```
# config

## encrytion key
`consul keygen > consul.key`

## config
same config on both node, change `encrypt` to key just generated, `auto_join` is hostname or IP of nodes

```
{
    "bootstrap_expect": 2,
    "client_addr": "0.0.0.0",
    "datacenter": "HOME",
    "data_dir": "/var/consul",
    "domain": "consul",
    "enable_script_checks": true,
    "dns_config": {
        "enable_truncate": true,
        "only_passing": true
    },
    "enable_syslog": true,
    "encrypt": "NAJ8GyIFy9DkSFnsuY4qCcKsWdW2M900Ux8sfgrpIcI=",
    "leave_on_terminate": true,
    "log_level": "INFO",
    "rejoin_after_leave": true,
    "server": true,
    "start_join": [
    	"consul1",
    	"consul2"
    ],
    "ui": true
}

```

## add service
### create new service
```
cat > /etc/systemd/system/consul.service << EOF
[Unit]
Description=Consul Startup process
After=network.target
 
[Service]
Type=simple
ExecStart=/bin/bash -c '/usr/local/bin/consul agent -config-dir /etc/consul.d/'
TimeoutStartSec=0
 
[Install]
WantedBy=default.target
EOF

```
### enable and start service

```
systemctl daemon-reload
systemctl start consul
/usr/local/bin/consul members
```

UI: http://<consul-IP>:8500/ui
