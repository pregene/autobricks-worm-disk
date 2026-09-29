# Install Autobricks WORM Disk

Use a package matching the destination Linux distribution and architecture.
Install with APT so its dependencies, including `cryptsetup` and `e2fsprogs`, are resolved:

```sh
sudo apt install ./autobricks-worm-disk-<version>-<os>-<os-version>-<arch>.deb
```

The first installation opens an interactive terminal interface. Prepare a
writable, unmounted dedicated partition. Select the partition, mount directory,
retention period, and access options. The default mount directory is
`/mnt/worm-disk`. LUKS is required and cannot be unchecked. TPM 2.0 is shown as a disabled
option and cannot be selected.

The installer displays the selected partition and its capacity before asking
for `YES`. Initialization erases its existing data. Enter or any other response
cancels initialization. An existing LUKS partition can also be selected,
regardless of which application created it. Confirming initialization replaces
its previous encryption and data with the selected storage format. No separate
signature-removal step is required.

The installer generates a LUKS password and saves it in
`/etc/ab-worm-disk/config.json` before initializing the disk. The directory is
root-owned with mode `0700`; the configuration file has mode `0600`. Protect any
backup of this file as a disk-unlock credential. Do not print it into logs or
include it in support attachments.

The `ab-worm-disk.service` service opens the configured LUKS volume, mounts the
WORM filesystem, and closes the mapping after unmounting both the public WORM
mount and its private ext4 mount. Check it with:

```sh
sudo systemctl status ab-worm-disk.service
sudo journalctl -u ab-worm-disk.service
```

## Reinstall or upgrade

Installing over an existing configuration preserves all settings and passwords,
without repeating the questionnaire or initializing the disk. The running
service is not automatically restarted. To run the newly installed binary:

```sh
sudo systemctl restart ab-worm-disk.service
```

An interrupted initialization also preserves its configuration and password.
Reinstalling does not retry destructive initialization automatically.

## Remove or purge

```sh
sudo apt remove autobricks-worm-disk
```

Removal stops the service and preserves configuration and disk-access
credentials. The disk and its contents are retained.

```sh
sudo apt purge autobricks-worm-disk
```

Purge warns that deleting the configuration removes disk-access information.
Only an exact `YES` deletes it. Enter, another response, or the absence of an
interactive terminal cancels deletion. The disk itself is never erased by purge.

After purge, a new installation displays the setup screens again and can
initialize the selected partition, including an existing LUKS partition, after
your explicit confirmation.

To reuse existing encrypted data instead of initializing it, restore a secure
backup of the configuration with its original ownership and permissions.
Without a valid unlock credential, existing encrypted data cannot be read;
installing the package does not recover a deleted password.

Use of unmodified source builds and redistribution are governed by
[LICENSE](LICENSE). The package includes the command, installation interface,
systemd service, documentation, and license notices.
