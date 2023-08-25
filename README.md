# S.M.A.R.T.-disk-monitoring-for-Prometheus text collector

Prometheus `node_exporter` `text_collector` for S.M.A.R.T disk values

The following dashboards are designed for this exporter:

https://grafana.com/grafana/dashboards/13654

## Purpose
This text collector is a customized version of the S.M.A.R.T. `text_collector` example from `node_exporter` github repo:
https://github.com/prometheus/node exporter/tree/master/text collector examples

## Requirements
- Prometheus
- `node_exporter`
  - text collector enabled for node exporter
- Grafana >= 7.3.6
- smartmontools >= 7.0

## Set up
To enable text collector set the following flag for `node_exporter`:
- `--collector.textfile.directory=/var/lib/node_exporter/textfile_collector`

To get an up to date version of smartmontools it could be necessary to compile it:
https://www.smartmontools.org/wiki/Download#Installfromthesourcetarball

- check by executing `smartctl --version`

- make smartmon.sh executable (`chmod +x smartmon.sh`)

- save it under `/usr/local/bin/smartmon.sh`

To enable the text collector on your system add the following as cronjob.
It will execute the script every five minutes and save the result to the `text_collector` directory.

Example for *UBUNTU* `crontab -e`:

`*/5 * * * * /usr/local/bin/smartmon.sh > /var/lib/node_exporter/textfile_collector/smart_metrics.prom`

Example for *FreeBSD* `crontab -e`:

`*/5 * * * * /usr/local/bin/smartmon.sh > /var/tmp/node_exporter/smart_metrics.prom`

### Number formatting

`smartmon.sh` uses system utilities that by default format numbers according to the localization settings configured for the system.

On systems using locales that format decimal numbers with the comma instead of the dot (e.g. `1,4` instead of `1.4`), node-exporter may fail to parse values, producing a log line such as:

```
ts=2022-11-25T11:55:00.384Z caller=textfile.go:227 level=error collector=textfile msg="failed to collect textfile data" file=smart_metrics.prom err="failed to parse textfile data from \"/var/lib/node_exporter/textfile_collector/smart_metrics.prom\": text format parsing error in line 21: expected float as value, got \"0,000000\""
```

This can be fixed in various ways; the easiest one consists of executing the script with the environment variable `LC_NUMERIC=C`:

```shell
LC_NUMERIC=C /usr/local/bin/smartmon.sh
```


## How to add specific S.M.A.R.T. attributes
If you are missing some attributes you can extend the text collector.
Add the desired attributes to `smartmon_attrs` array in `smartmon.sh`.

You get a list of your disks privided attributes by executing:
`sudo 	smartctl -i -H /dev/<sdx>`
`sudo 	smartctl -A /dev/<sdx>`

## How to force script to collect information only for the chosen disks
If you are certain of what disks you want to monitor and you don't want to rely on the smart --scan mechanism,
you can uncomment and set the `FORCED_DEVICE_LIST` variable:
```
FORCED_DEVICE_LIST=$(cat << EOF
/dev/sg3|scsi
/dev/sg4|scsi
/dev/sda|sat
/dev/sdb|sat
EOF
)
```

Device type can be probed with `smartctl -d test /dev/sdX` command.
