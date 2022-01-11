# Baseline

An Ansible role that installs, configures and manages a baseline for EL 8.

 - selinux
 - resolv.conf
 - a neat 'history' configuration
 - set nice colors for the terminal
 - time via chrony
 - system locale
 - default editor
 - motd
 - configures sshd parameters
 - disabled firewalld
 - it adds repositories
 - default packages
 - sysctl configuration

## SElinux

By default, SElinux is configured to disabled, a system reboot is executed at the end of the role if that a change has happened.:

```yaml
# reboot after a config change of selinux
selinux_state: disabled
selinux_reboot: true
```

## Resolv.conf

The resolv.conf is set, and the file `/etc/NetworkManager/conf.d/90-dns-none.conf` is set to none by default.
The NetworkManager service is then restarted. This is done by default.

Define your own with:

```yaml
nameservers:
  - 1.1.1.1
search_domain: my_search_domain.com
```

## History
A configuration is placed in /etc/profile to let the history command show:

```bash
user@machine:~$ history
    1  2022-01-30 12:40:59 my-command
```

## Time

```yaml
time: true
timezone: Europe/Amsterdam
```

## System locale

```yaml
locale: true
locale_lang: 'en_US.UTF-8'
locale_language: 'en_US.UTF-8'
```

## Editor

A basic editor is set:

```yaml
editor: true
editor_application: vim
```
It installs the package and appends the following in `/etc/profile.d/<app_name>.sh`

```bash
export VISUAL=vim
export EDITOR=vim
```

## MOTD

A motd is set.

```yaml
motd: true
motd_owner: someone
```

Results in:

```bash
## This machine is owned by someone ##
--------------------INFO-------------------
 - Hostname        : vm-local-1
 - Uptime          : up 1 hours, 1 minutes
 - Logged in users : 1
 - Release         : Rocky Linux release 8.4 (Green Obsidian)
--------------------------------------------
```

## SSHD

The file /etc/sshd/sshd_config is edited with lineinfile and validated before restarting sshd.

```
sshd: true
sshd_parameters:
  - line: 'PermitRootLogin no'
    regexp: '^PermitRootLogin'
    state: present
```

## Firewalling

This role disabled firewalld by default.

```yaml
firewall: true
```

I haven't wrote any configuration to handle further configuration because that would require it's own role.

## Repositories

Repositories can be managed via this role as well.

```yaml
repos: true
repos_epel: true
repos_custom:
# example of a custom repo
  - name: epel-something
    description: "Repo for EL $releasever - $basearch"
    baseurl: "https://download.fedoraproject.org/pub/epel/$releasever/$basearch/"
    mirrorlist: "" # can be left empty
    enabled: 'yes'  # either yes or no
    state: present
    gpgcheck: yes # can be left empty, default is no
    gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8 # can be left empty
    includepkgs: vim # can be left empty
```

When `repos_epel` is true, the epel-release is installed.

## Default pacakges

Default packages are installed.

```yaml
packages: true
packages_to_install:
  - bc
  - curl
  - git
  - glibc-all-langpacks
  - iotop
  - langpacks-nl
  - logwatch
  - lsof
  - mailx
  - mlocate
  - nano
  - nc
  # - net-snmp # installs mariadb-connector. Not desired when using db's
  - net-tools
  - openssh
  - openssh-server
  # - postfix
  - psmisc
  - rsync
  - socat
  - strace
  - sudo
  - sysstat
  - tcpdump
  - telnet
  - tmux
  - tree
  - unzip
  - vim
  - wget
  - whois
  - xinetd
  - yum-utils
  - zip
```

## Sysctl

Set custom sysctl settings.

```yaml
sysctl: true
sysctl_config:
  - name: kernel.panic
    value: '1'
    file: /etc/sysctl.conf
```
