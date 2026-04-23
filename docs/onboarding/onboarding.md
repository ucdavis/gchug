# Getting started

## Basic command line info
Finish the [command line tutorial](https://labex.io/lesson/the-shell) on linuxjourney

## Setting up a Hive account
Go to [HiPPO](https://hippo.ucdavis.edu/clusters) and make an account.

See [ssh_keys.md](./ssh_keys.md) for more information about setting up ssh keys

## Logging in and navigating

Once you have made your account on [HiPPO](https://hippo.ucdavis.edu/clusters), you should be able to log into Hive. You username and password will be the same as your kerberos ID login credentials.

You can log into Hive by opening the terminal (command line) and typing in `ssh username@hive.hpc.ucdavis.edu`. If you have set up an ssh key, you won't need to enter a password

```sh
# from your computer
ssh username@hive.hpc.ucdavis.edu

# enter password if prompted
# expository hive text will be printed

# Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.5.0-45-generic x86_64)
#  _   _   _   _
# / \_/ \_/ \_/ \             Welcome to Hive.
# \_/ \_/ \_/ \_/
# / \_/  _   _ _              Documentation available # here:
# \_/ \ | | | (_)_   ____     # https://docs.hpc.ucdavis.edu/
# / \_/ | |_| | \ \ / /_ \
# \_/ \ |  _  | |\ V / __/    For more information:
# / \_/ |_| |_|_| \_/\___|    https://hpc.ucdavis.edu
# \_/ \_  .hpc.ucdavis.edu
# / \_/ \_                    Need help? hpc-# help@ucdavis.edu
# \_/ \_/ \_/
# / \_/ \_/                   Maintenance changes:
# \_/ \_/                     # https://docs.hpc.ucdavis.edu/maintenance/december-2025/
```

When you first log into hive, you will get sent to `/home/username`

```sh
# from your computer
ssh username@hive.hpc.ucdavis.edu

# enter password if prompted
# expository hive text will be printed

pwd
# /home/username
```

You can then move to your lab's directory

```sh
cd /quobyte/pi_namegrp
```
