# ssh-to-actions
A GitHub Action that setups up a reverse SSH tunnel allowing you to SSH into the runner box from any internet-accessible machine.

## How it works

We start by setting up an SSH reverse tunnel from the Actions host to your internet-accessible host. The tunnel tells your accessible host to route ssh traffic on localhost:22222 to the Actions host. Once the tunnel is configured, we just ssh to the local port 22222 and the tunnel takes care of everything else.

There are a couple different auth points. The Actions server has to auth with your accessible host when we set up the tunnel. We use SSH keys for this since I wouldn't want password auth on a public SSH server. We also have to auth into the Actions server when SSHing. We use a password for this since the runner is ephemeral, and should only be accessible via the tunnel. This could be configured to use another SSH pub/private key pair.

## Setup

### Configure Secrets

The action depends on a number of secrets to be set in settings/secrets (or via the API).

`ssh_public_key`/`ssh_private_key` - The GitHub Actions runner must authenticate with your internet-accessible machine. To do so, we use a SSH public/private key pair. Please don't use existing keys, generate new ones!

`reachable_host` - The host you'll be using to SSH into the actions runner. Note this _must_ be internet-accessible. E.g. `root@111.111.111.111`

`password` - We reset the GitHub Actions runner password and then use the password to auth into the box when SSHing into it. Set to whatever you like.

### Run action

Once your secrets are setup, run the GitHub Action. If all goes well, it'll hang on the "Connect to remote server" step. This means SSH has configured the tunnel and is waiting for traffic.

On your local machine, run `ssh -p 22222 runner@localhost`. Port 22222 is where we configured the tunnel, and `runner` is the hostname on the Actions server. This may error with a Offending ECDSA key error, which is expected since the runner is ephemeral and will be a new machine every time we try to connect. Simply run `ssh-keygen -f "/root/.ssh/known_hosts" -R [localhost]:22222` to clear up the error, and try again.

When prompted for a password, use the password you set when configuring the secrets.

After entering a password, you should be setup with a SSH connection to the runner machine :)

![](https://i.imgur.com/yGSsRgY.png)
