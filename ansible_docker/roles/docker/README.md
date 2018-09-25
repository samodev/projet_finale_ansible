# ansible-role-docker-ce

[![Ansible Role](https://img.shields.io/ansible/role/17533.svg)](https://galaxy.ansible.com/haxorof/docker-ce/)
[![GitHub tag](https://img.shields.io/github/tag/haxorof/ansible-role-docker-ce.svg)](https://github.com/haxorof/ansible-role-docker-ce)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/haxorof/ansible-role-docker-ce/blob/master/LICENSE)
[![Build Status](https://travis-ci.com/haxorof/ansible-role-docker-ce.svg?branch=master)](https://travis-ci.com/haxorof/ansible-role-docker-ce)

Installs and configures Docker CE (Community Edition) on CentOS, Fedora, Ubuntu, Debian, RHEL<sup>†</sup>, and Linux Mint<sup>†</sup>.

<sup>†</sup> NB: Docker does _not_ officially support Docker CE on this distribution.

## Changelog

See changelog [here](https://github.com/haxorof/ansible-role-docker-ce/blob/master/CHANGELOG.md)

## Ansible Compatibility

* `2.4` or later

For this role to support multiple Ansible versions it is not possible to avoid all Ansible deprecation warnings. Read Ansible documentation if you want to disable [deprecation warnings](http://docs.ansible.com/ansible/latest/reference_appendices/config.html#deprecation-warnings).

If you are using Ansible version `2.3` you cannot go higher than version `1.5.0` of this role.

## Requirements

No additional requirements.

## Role Variables

Variables related to this role are listed below:

```yaml
# Daemon configuration (https://docs.docker.com/engine/reference/commandline/dockerd/)
# Example:
# docker_daemon_config:
#   experimental: true
docker_daemon_config: {}
# Docker daemon options
#  Docker daemon is configured with '-H fd://' by default in Ubuntu/Debian which cause problems.
#  https://github.com/moby/moby/issues/25471
docker_daemon_opts: ''
# Map of environment variables to Docker daemon
docker_daemon_envs: {}
# List of additional service configuration options for systemd
# Important! Configuring this can cause Docker to not start at all.
docker_systemd_service_config: []
# To compensate for situation where Docker daemon fails because of usermod incompatibility.
# Ensures that 'dockremap:500000:65536' is present in /etc/subuid and /etc/subgid.
# Note! If userns-remap is set to 'default' in docker_daemon_config this config will be unnecessary.
docker_bug_usermod: false
# Enable auditing of Docker related files and directories
docker_enable_audit: false
# Enable Docker CE Edge
docker_enable_ce_edge: false
# Set `MountFlags=slave`
#  https://github.com/haxorof/ansible-role-docker-ce/issues/34
docker_enable_mount_flag_fix: yes
# Always ensure latest version of Docker CE
docker_latest_version: true
# Docker package to install. Change this if you want a specific version of Docker
docker_pkg_name: docker-ce
# If below variable is set to true it will remove older Docker installation before Docker CE.
docker_remove_pre_ce: false
```

## Dependencies

None.

## Example Playbook

Following sub sections show different kind of examples to illustrate what this role supports.

### Simplest

```yaml
- hosts: docker
  roles:
    - role: haxorof.docker-ce
```

### Configure Docker daemon to use proxy

```yaml
- hosts: docker
  vars:
    docker_daemon_envs:
      HTTP_PROXY: http://localhost:3128/
      NO_PROXY: localhost,127.0.0.1,docker-registry.somecorporation.com
  roles:
    - haxorof.docker-ce
```

### On the road to CIS security compliant Docker engine installation

This minimal example below show what kind of role configuration that is required to pass the [Docker bench](https://github.com/docker/docker-bench-security) checks.
However this configuration setup devicemapper in a certain way which will create logical volumes for the containers. Simplest is to have at least 3 GB of free space available in the partition. Since Docker v17.06 it is possible to just set the storage option `dm.directlvm_device` to make Docker create the necessary volumes:

```yaml
- hosts: docker
  roles:
    - role: haxorof.docker-ce
      docker_enable_audit: true
      docker_daemon_config:
        icc: false
        init: true
        userns-remap: default
        disable-legacy-registry: true
        live-restore: true
        userland-proxy: false
        log-driver: journald
        storage-driver: devicemapper
        storage-opts:
          - "dm.directlvm_device=/dev/sdb1"
```

Because the configuration above requires Linux user namespaces to be enabled then additional GRUB arguments might be needed. Example below show one example what changes that might be needed and reboot of the host is required for the changes to take full affect.

```yaml
# https://success.docker.com/article/user-namespace-runtime-error

- hosts: docker
  roles:
    - role: jtyr.grub_cmdline
      vars:
        grub_cmdline_add_args:
          - namespace.unpriv_enable=1
          - user_namespace.enable=1
      become: yes
  tasks:
    - name: set user.max_user_namespaces
      sysctl:
        name: user.max_user_namespaces
        value: 15000
        sysctl_set: yes
        state: present
        reload: yes
      become: yes
```

## License

This is an open source project under the [MIT](https://github.com/haxorof/ansible-role-docker-ce/blob/master/LICENSE) license.
