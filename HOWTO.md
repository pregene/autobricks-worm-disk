# Use the WORM disk

Applications access files through the configured mount directory, normally
`/mnt/worm-disk`.

```sh
mkdir /mnt/worm-disk/example
printf 'first record\n' > /mnt/worm-disk/example/events.log
printf 'next record\n' >> /mnt/worm-disk/example/events.log
cat /mnt/worm-disk/example/events.log
```

Use a new filename for the initial write. Repeating `>` on an existing file
attempts truncation and is refused. Existing data cannot be overwritten, files
cannot be renamed, and files cannot be deleted before their retention deadline. Integrity and retention metadata
are internal and are not exposed as `.meta` files.

## Read checksum metadata

Use the mounted file path, without sudo:

```sh
ab-worm-disk checksum /mnt/worm-disk/example/events.log
```

The command prints JSON containing `format_version`, `created_at`,
`retain_until`, `lock_offset`, and `checksum`. Times use the storage retention clock (Linux CLOCK_BOOTTIME), not calendar time;
`lock_offset` is the committed byte count, and `checksum` is its stored SHA-256
digest. The command reads a consistent metadata snapshot without rehashing file
contents. The copyright and version banner goes to stderr, leaving stdout as
JSON. Internal hash checkpoint state is not exposed.

Metadata is read through the mounted filesystem; users do not need the raw
disk or the LUKS configuration password. The file must be readable by the caller.
For a root-owned service mount, keep **Allow other users** enabled (the installer
default). A mount without that option remains restricted to its mounting user.
Metadata cannot be changed or removed through this interface. `.meta` files
remain absent from directory listings and direct path access.

The retention period applies to newly created files. Existing files retain their
original deadlines. Access by other users is controlled by the installation's
allow-other setting and filesystem permissions; WORM restrictions still apply.

```sh
sudo systemctl stop ab-worm-disk.service
sudo systemctl start ab-worm-disk.service
```

Stopping the service unmounts the filesystem and closes its LUKS mapping when
encryption is enabled. If shutdown fails, inspect the service journal before
starting it again. See [INSTALL.md](INSTALL.md) for installation, upgrades, and
configuration preservation during removal.
