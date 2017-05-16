ansible-role-lsyncd-multimaster
==========================

This role will let you replicate multiple directories to multiple targets, using lsyncd 2.x in
a not-the-most-elegant multi-master setup. It however, works.

The premise of lsyncd is that it uses kernel inotify to detect file system changes combined with
a LUA configuration allowing for an endless set of event handling possibilities. The most common
usage is by invoking rsync to sync data between sets of hosts.

This setup requires passwordless key ssh between all hosts in the lsyncd_fanout_group. Optionally,
one can provide ssh options that strictly assoicate to a private key which is command restricted
to rsync:
https://www.rfxn.com/irsync-limiting-passwordless-ssh-keys/

## Variables

- `lsyncd_fanout_delay`: delay between file syncs (default: 5)
- `lsyncd_fanout_directories`: directories to sync (required, default: false, i.e.
none)
- `lsyncd_fanout_max_processes`: max concurrent sync processes (default: 5)
- `lsyncd_fanout_interface`: what network interface do we use to communicate (ipv4 address grabbed and bound as listener)
- `lsyncd_fanout_group`: name of ansible hosts group containing hosts to sync across 
- `lsyncd_fanout_ssh_options`: string of options that are passed to ssh
- `lsyncd_fanout_rsync_delete`: true/false if rsync should delete data between hosts (default: false); when false, rsync will perform update-only operations which will overwrite and update-contents of existing files but not remove any files on deletion
- `lsyncd_fanout_rsync_options`: extended options for rsync calls (default: -lturs)

- `inotify_max_user_watches`: maximum amount of per-user watched objects [read: files and directories] (default: 524288)
- `inotify_max_user_instances`: maximum amount of per-user inotify instances [read: number of unique paths monitored] (default: 256)
- `inotify_max_queued_events`: maximum amount of queued inotify events waiting for polling from a watch instance [read: dont set fanout_delay too high or queued events will max out] (default: 16384)


## Usage

- hosts: network
  roles:
    - role: ansible-role-lsyncd-multimaster
      lsyncd_fanout_group: 'webservers'
      lsyncd_fanout_interface: 'em1'
      lsyncd_fanout_directories:
        - '/data'

This will sync directory `/data` on all hosts in the 'webservers' group, using the IP address on `em1`. You can
set multiple directories if you have several of them to sync.
