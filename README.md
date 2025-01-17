# Jellyfin Config Backup (adjusted for lscr.io/linuxserver/jellyfin:latest)
### **This script requires Python 3 and is designed for Synology DSM7** 
This script backs up a Jellyfin server configuration running in a Docker container on Synology DSM7. It is designed only to back up the configuration files in the event of corruption or deletion. It *is not* meant to backup your entire Jellyfin media folder. It has only been tested with the Python3.8 that's default for DSM7.

This works best if you have a static data folder for your Jellyfin container. Otherwise, you'll need to find out the hash for your data container. Additionally updating the container without a static data location will mean the hash will change, and you'll need to update your script. So it's always recommended to use a static data folder.

It's also recommended that keep both your static Jellyfin config and script locations separate and *outside* of network shares (`'/volume1/docker/jellyfin/config'` and  `'~/scripts/'` respectively), simply for security and stability reasons. Having your Jellyfin config accessible via network share is one more point of failure. `'/volume1/.scripts'` is a non-standard recommended location for a reason. As well, if anything in your Jellyfin config gets deleted or changed by anything but the Jellyfin server and things may be borked and your server may.  

Credit to [Gabisonfire](https://github.com/Gabisonfire/emby_backup) for the original emby_backup from which this is based.

## How It Works

The script is pretty simple. It copies a slightly-customizable list of Jellyfin config files to a temporary directory, compresses the directory and saves that a .zip file in the destination folder. It then proceeds to delete old backups so that you have a customizable number remaining.

## Script Installation

[Different methods of installation can be found here](https://github.com/zang74/jellyfin_config_backup/wiki/Installation "Installation instructions.")

## Getting the Synology to use it

[There's an easy visual guide in the Wiki](https://github.com/zang74/jellyfin_config_backup/wiki/Setting-up-Synology-Task-Scheduler)

## Usage

- "-v", "--version",
  - Gives the current version of the script.
- "-c", "--configpath",
  - Jellyfin's data container config path. It defaults to /volume1/docker/jellyfin/config".
- "-d", "--destination",
  - Destination folder for backups. This is required.
- "-k", "--keep",
  - [Optional] Number of saved archives. If you are backing up metadata (see below) it's recommended to keep this number relatively low to avoid filling up your Synology with unnecessarily large files. The default is five.
- "-o", "--other",
  - [Optional] Additional file(s) to include. Can be repeated per file.
- "-m", "--metadata",
  - [Optional] This will also back up the Jellyfin metadata folder and is off by default. Use this option with caution, as including metadata can be both slow and will dramatically increase the size of the backup files. A better, faster option is storing metadata alongside media files in the form of .NFO and associated image files and doing regular incremental backups of your media file directories.
- "-l", "--logfile",
  - [Optional] /path/to/logfile. Change this if you want logs in a more easily accessible location, such as on one of the NAS' SMB shares. The default save location is /var/log/jellyfinbackup.log
- "-i", "--ignoredevicehash",
  - [Optional] Skip backing up the device.txt hash file. This might be useful if moving the config from one machine to another.
## Examples

```
python3 /volume1/.scripts/jellyfin_config_backup.py -c "/path/to/jellyfin/config/" -d "/path/to/backup/destination/" -o "/etc/additional_file" -o "/some/other/file" -m
python3 /volume1/.scripts/jellyfin_config_backup.py --configpath "/path/to/jellyfin/config/" --destination "/path/to/backup/destination/" --logfile "/path/to/network/share" --ignoredevicehash
python3 /volume1/.scripts/jellyfin_config_backup.py -d "/path/to/backup/destination/" -k 3
```
## Warning

I'm serious about avoiding a backup of the metadata directory. That's why it's off by default. On my own system, what is a 62MiB file easily balloons to almost 3GiB if I add metadata. If you're backing up even a couple of times a week, this can easily become a big problem, shrinking your filespace dramatically and repeatedly creating huge files of almost entirely the same data.

## Note

If you've set up Jellyfin without static data directories, your config and cache folders will exist in random-hash-named directories under volume1/@docker/containers. Every time your Jellyfin Docker Container needs to be updated, a new random-hash-named directory will be created leaving the old one sitting there to rot. This will break the functionality of both Jellyfin Config Backup *and* the Jellyfin Media Server. Every new update to Jellyfin will act like an out-of-the-box server. If you plan on updating Jellyfin, you should set it up with static directories.

[A simple guide can be found here](https://mariushosting.com/how-to-install-jellyfin-on-your-synology-nas/)
