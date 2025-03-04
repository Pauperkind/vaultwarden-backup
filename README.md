# Vaultwarden Backup

A simple cron powered backup image for [vaultwarden](https://github.com/dani-garcia/vaultwarden).

:warning: Since Bitwarden_RS has been [renamed](https://github.com/dani-garcia/vaultwarden/discussions/1642#discussion-3344543) to Vaultwarden I also renamend this project to vaultwarden-backup. I will continue pushing the Image also to the old Docker repository for convenience. However you should also switch to the new Docker repository <https://hub.docker.com/r/bruceforce/vaultwarden-backup> if you find some time. To do so just replace the `bruceforce/bw_backup` image by `bruceforce/vaultwarden-backup`.

## Why vaultwarden-backup?

You might ask yourself "Why should I use a container for backing up my vaultwarden files if I can just include them in my regular backup?". One caveat of using regular backup software for database is, that you shoud always stop your database server before you make a backup or you will risk data loss. To prevent this a proper backup command for your database should be used.

Of course you could just create a cron job on your host with something like `sqlite3 "$VW_DATABASE_URL" ".backup '$BACKUP_FILE_DB'"` and back up the additional files and folders (like the attachments folder), using your preferred backup solution.

However on some systems you are not able to add cronjobs by yourself, for example common NAS venderos don't allow this. That's why this image exists. Additionally it also includes the most important files and puts them in a `tar.gz` archive from where on your regular backup software could handle this files.

## Which files are included in the backup?

By default all files that are recommended to backup by the official Vaultwarden wiki <https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault> are backed up per default.

## Usage
Since version v0.0.7 you can always use the `latest` tag, since the image is build with
multi-arch support. Of course you can always use the version tags `vx.y.z` to stick
to a specific version. Note however that there will be no security updates for the
alpine base image if you stick to a version.

Make sure that your **vaultwarden container is named `vaultwarden`** otherwise 
you have to replace the container name in the `--volumes-from` section of the `docker run` call.

### Automatic Backups 
A cron daemon is running inside the container and the container keeps running in background.

Start backup container with default settings (automatic backup at 5 am)
```sh
docker run -d --restart=always --name vaultwarden-backup --volumes-from=vaultwarden bruceforce/vaultwarden-backup
```

Example for hourly backups
```sh
docker run -d --restart=always --name vaultwarden-backup --volumes-from=vaultwarden -e CRON_TIME="0 * * * *" bruceforce/vaultwarden-backup
```

Example for backups that delete after 30 days
```sh
docker run -d --restart=always --name vaultwarden --volumes-from=vaultwarden -e TIMESTAMP=true -e DELETE_AFTER=30 bruceforce/vaultwarden-backup
```

### Manual Backups
You can use the crontab of your host to schedule the backup and the container will only be running during the backup process.

```sh
docker run --rm --volumes-from=vaultwarden bruceforce/vaultwarden-backup manual
```

If you want the backed up file to be stored outside the container you have to mount
a directory by adding `-v <PATH_ON_YOUR_HOST>:<PATH_INSIDE_CONTAINER>`. The complete command could look like this

```sh
docker run --rm --volumes-from=vaultwarden -e UID=0 -e BACKUP_DIR=/myBackup -e TIMESTAMP=true -v $(pwd)/myBackup:/myBackup bruceforce/vaultwarden-backup manual
```

Keep in mind that the commands will be executed *inside* the container. So `$BACKUP_DIR` can be any place inside the container. Easiest would be to set it to `/data/backup` which will create the backup next to the original database file.

### Restore

There is no automated restore process to prevent accidential data loss. So if you need to restore a backup you need to do this manually by following the steps below (assuming your backups are located at `./backup/` and your vaultwarden data ist located at `/var/lib/docker/volumes/vaultwarden/_data/`)

```sh
# Delete any existing sqlite3 files
rm /var/lib/docker/volumes/vaultwarden/_data/db.sqlite3*

# Copy the database to the vaultwarden folder
cp ./backup/db.sqlite3 /var/lib/docker/volumes/vaultwarden/_data/db.sqlite3

# Extract the additional folder from the archive
tar -xzvf ./backup/data.tar.gz -C /var/lib/docker/volumes/vaultwarden/_data/
```

## Environment variables
| ENV                         | Description                                                                         |
| --------------------------- | ----------------------------------------------------------------------------------- |
| BACKUP_ADD_DATABASE [^3]    | Set to `true` to include the database itself in the backup                          |
| BACKUP_ADD_ATTACHMENTS [^3] | Set to `true` to include the attachments folder in the backup                       |
| BACKUP_ADD_CONFIG_JSON [^3] | Set to `true` to include `config.json` in the backup                                |
| BACKUP_ADD_ICON_CACHE [^3]  | Set to `true` to include the icon cache folder in the backup                        |
| BACKUP_ADD_RSA_KEY [^3]     | Set to `true` to include the RSA keys in the backup                                 |
| BACKUP_ADD_SENDS [^3]       | Set to `true` to include the sends folder in the backup                             |
| BACKUP_DIR                  | Seths the path of the backup folder *inside* the container                          |
| BACKUP_DIR_PERMISSIONS      | Sets the permissions of the backup folder (**CAUTION** [^1]). Set to -1 to disable. |
| CRONFILE                    | Path to the cron file *inside* the container                                        |
| CRON_TIME                   | Cronjob format "Minute Hour Day_of_month Month_of_year Day_of_week Year"            |
| DELETE_AFTER                | Delete old backups after X many days. Set to 0 to disable                           |
| TIMESTAMP                   | Set to `true` to append timestamp to the backup file                                |
| GID                         | Group ID to run the cron job with                                                   |
| HEALTHCHECK_URL             | Set a healthcheck url like <https://hc-ping.com/xyz>                                |
| LOG_LEVEL                   | DEBUG, INFO, WARNING, ERROR, CRITICAL are supported                                 |
| LOG_DIR                     | Path to the logfile folder *inside* the container                                   |
| LOG_DIR_PERMISSIONS         | Sets the permissions of the logfile folder. Set to -1 to disable.                   |
| TZ                          | Set the timezone inside the container [^2]                                          |
| UID                         | User ID to run the cron job with                                                    |
| VW_DATA_FOLDER [^4]         | Set the location of the vaultwarden data folder *inside* the container              |
| VW_DATABASE_URL [^4]        | Set the location of the vaultwarden database file *inside* the container            |
| VW_ATTACHMENTS_FOLDER [^4]  | Set the location of the vaultwarden attachments folder *inside* the container       |
| VW_ICON_CACHE_FOLDER [^4]   | Set the location of the vaultwarden icon cache folder *inside* the container        |

For default values see [src/opt/scripts/set-env.sh](src/opt/scripts/set-env.sh)

[^1]: The permissions should at least be 700 since the backup folder itself gets the same permissions and with 600 it would not be accessible.
[^2]: see <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones> for more information
[^3]: See <https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault> for more details
[^4]: See <https://github.com/dani-garcia/vaultwarden/wiki/Changing-persistent-data-location> for more details

## FAQ
### I get an error like "unable to open database file"
`Error: unable to open database file` is most likely caused by permission errors.
Note that sqlite3 creates a lock file in the source directory while running the backup.
So source *AND* destination have to be +rw for the user. You can set the user and group ID
via the `UID` and `GID` environment variables like described above.

### Database is locked
`Error: database is locked` is most likely caused by choosing a backup location that is *not* on the same filesystem as the vaultwarden database (like a network filesystem).

Vaultwarden, when started with default settings, uses WAL (write-ahed logging). You can verify this by looking for a `db.sqlite3-wal` file in the same folder as your original database file. According to the official SQLite docs WAL will cause issues in network share scenarios (see https://www.sqlite.org/wal.html):

> All processes using a database must be on the same host computer; WAL does not work over a network filesystem.

Basically there are two workarounds for this issue

1. Choose a local target for your backup and then use some other tool like `cp` or `rsync` to copy the backup file to your network filesystem.
2. Disable WAL in Vaultwarden. You can find a guide here (https://github.com/dani-garcia/vaultwarden/wiki/Running-without-WAL-enabled).

### Date Time issues / Wrong timestamp
If you need timestamps in your local timezone you should mount `/etc/timezone:/etc/timezone:ro` and `/etc/localtime:/etc/localtime:ro`
like it's done in the [docker-compose.yml](docker-compose.yml). An other possible solution is to set the environment variable accordingly (like  `TZ=Europe/Berlin`) 
(see <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones> for more information).

**Attention** if you are on an ARM based platform please note that [alpine](https://alpinelinux.org/) is used as base image for this project to keep things small. Since alpine 3.13 and above it's possible that you will end up with a container with broken time and date settings (i.e. year 1900). This is a known problem in the alpine project (see [Github issue](https://github.com/alpinelinux/docker-alpine/issues/141) and [solution](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.13.0#time64_requirements)) and there is nothing I can do about it. However in the [alpine wiki](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.13.0#time64_requirements) a solution is being proposed which I also tested tested on my raspberry pi. After following the described process it started working again as expected. If you still experience issues or could for some reason not apply the aforementioned fixes please feel free to open an issue.

### Why is the container started by the root user?
The  main reason to build this image was to allow users to run sheduled tasks where their host OS does not allow them to do so, or where they want a "portable" way of using a scheduled tasks without relying on host OS mechanisms.

Since `crond` *must* be run as root user there is no way to start this container as a non-root user while using cron. I'm aware that there are other task schedulers like [supercronic](https://github.com/aptible/supercronic) which allow to run without root privileges but I want to stay with the standard and established cron system for the time being.

### Why sh is used instead of bash
Alpine by default comes without bash installed. Since the pre-installed `ash` shell is suitable for the tasks of this image and comes with no need to install additional tools like bash, `/bin/sh` is used as shell. The scripts also aims to be POSIX compliant which should make a switch of the base image fairly easy, if needed.