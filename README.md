# rpi-koray-syncer-0

This project is a config definition of a locally hosted gitea instance, used to mainly sync Github repositories and have a local
backup.

## Setup

You need to create a user on the host called git (used for ssh access to the repos).

UID and GID 2002 is used. You can run those commands:

```bash
sudo groupadd -g 2002 git
sudo useradd git -u 2002 -g 2002 -m -s /bin/bash
```

You can now create the host directory and chown it to this user:

```bash
sudo mkdir /mnt/gitea
sudo chown git:git /mnt/gitea
```

Then, create the gitea host ssh key (still on the host):

```bash
sudo -u git ssh-keygen -t ed25519 -C "Gitea Host Key"
```

Add the public key to authorized keys, still on the host. This is necessary for the ssh passthrough and because the authorized_keys
file is mounted to the container.

```bash
sudo -u git cat /home/git/.ssh/id_ed25519.pub | sudo -u git tee -a /home/git/.ssh/authorized_keys
sudo -u git chmod 600 /home/git/.ssh/authorized_keys
```

Create a gitea shim that will be used to pass on git commands via ssh to the container:

```bash
cat <<"EOF" | sudo tee /usr/local/bin/gitea
#!/bin/sh
ssh -p 2222 -o StrictHostKeyChecking=no git@127.0.0.1 "SSH_ORIGINAL_COMMAND=\"$SSH_ORIGINAL_COMMAND\" $0 $@"
EOF
sudo chmod +x /usr/local/bin/gitea
```
