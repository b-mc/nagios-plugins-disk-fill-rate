# Nagios plugin for predicting a disk fill

## Description
The plugin tries to predict a disk full scenario long before the disk actually fills up.

The disk-fill-rate plugin will check the _rate_ at which the
disks are being filled. If it is consistently being filled at a rate
which indicates that (if the rate is sustained) a disk fill up will happen
within a configured number of hours, it will raise a warning or a critical.

## How it works
The cron job `disk_fill_rate` runs every minute and keeps a log of how much free space is available. It keeps that in a text file, one line per mounted volume.

The `check_disk_fill_rate` plugin then reads the last few entries
(five minutes) of this log and calculates for each volume how fast
the free space is dwindling. It uses the result with the "slowest" rate
(i.e. the most conservative) and compares that with the thresholds.
The response is _warning_ or _critical_ if any disk is filling up within the configured thresholds, otherwise normal.

## What has changed in the fork?
- added input arguments
- added some safeguards and preconditions
- added debugging
- excluded overlay filesystems from the calculation
- code refresh and linting

## Configuration
### Script
```check_disk_fill_rate -w MINUTES -c MINUTES```
### Cron job entry
```* * * * * user /path/to/disk_fill_rate```
### NRPE command
```command[check_disk_fill_rate]=/usr/lib64/nagios/plugins/check_disk_fill_rate -w '$ARG1$' -c '$ARG2$'```
### Nagios service
```
define service{
        use                     generic-service
        host_name               your_host
        service_description     Disk Fill Rate
        check_command           check_nrpe!check_disk_fill_rate!w_threshold!c_threshold
        check_interval          60
        retry_interval          30
}
```