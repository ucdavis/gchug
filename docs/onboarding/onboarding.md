# Getting started

## Basic command line info
Finish the [command line tutorial](https://labex.io/lesson/the-shell) on linuxjourney

## Setting up a Hive account
Go to [HiPPO](https://hippo.ucdavis.edu/clusters) and make an account.

See [ssh_keys.md](./ssh_keys.md) for more information about setting up ssh keys

## Logging in and navigating

Once you have made your account on [HiPPO](https://hippo.ucdavis.edu/clusters), you should be able to log into Hive. You username and password will be the same as your kerberos ID login credentials.

You can log into Hive by opening the terminal (command line) and typing in `ssh username@hive.hpc.ucdavis.edu`. If you have set up an ssh key, you won't need to enter a password. When you first log in, you will be sent to `/home/zjamal`

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
