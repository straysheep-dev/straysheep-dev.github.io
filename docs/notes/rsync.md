---
title: "rsync"
icon: material/sync-circle
draft: false
#date:
#  created: 2025-02-22
#  updated: 2025-12-14
categories:
  - rsync
  - how-to
  - bash
  - cli
  - linux
  - unix
---

# :material-sync-circle: rsync

The fast, versatile, remote (and local) file-copying tool. It operates using deltas and by looking at properties to only update parts of files that have changed, making it incredibly efficient and invaluable as a part of a regular or scheduled backup operation. I have used this tool to ensure local, remote, and external copies of directories are in sync, or to identify what has changed.

<!-- more -->

## Practical Examples

### rsync from Host to Dev VM

First obtain the IP address of the VM.

=== "QEMU"

    ```bash
    # Tab completion of vm-name works
    virsh domifaddr <vm-name>
    ```

=== "Hyper-V"

    ```powershell
    # TO DO

    ```

Then execute the commands either as a bash script or through bash itself to perform the following:

- `-arv` archives permissions and unix file properties, recursively, and is verbose about rsync operations
- `--delete` will remove any files or data in the dest path that are no longer present in the src path
- `--safe-links` will not follow symlinks pointing outside of the tree
- `--exclude='<string>'` can be used multiple times to ignore any files and folders with `<string>` in their path
- `-e "<ssh-args>"` can be used to make additional SSH arguments, in the example below, jump across a firewall VM

Ultimately this will sync the folders `straysheep-dev.github.io`, `packer-configs`, and `ansible-configs` to `/home/user1/src/` on the remote machine. Without trailing slashes, the folders themselves are sync'd. If you included trailing slashes on the source paths, the **contents** of the folders would be sync'd. This would put the contents of all three folders into `~/src` on the remote machine, instead of keeping them in separate folders.

```bash
rsync -arv \
--delete \
--safe-links \
--exclude='.git/' \
--exclude='.cache' \
-e "ssh -J admin@<firewall-vm-ip>:2222" \
./straysheep-dev.github.io \
./packer-configs \
./ansible-configs \
user1@<vm-ip>:/home/user1/src/
```