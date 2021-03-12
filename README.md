# Pre-Kubernetes Ansible Playbook for Raspberry Pi

This repository contains the Ansible playbook that I use to configure the nodes in my [Raspberry Pi cluster](https://heathharrelson.github.io/posts/raspberry-pi-cluster-kubeadm/) before installing Kubernetes using [k3s-io/k3s-ansible](https://github.com/k3s-io/k3s-ansible) or my [heathharrelson/k8s-containerd-ansible](https://github.com/heathharrelson/k8s-containerd-ansible) playbook.

I built this using [Ubuntu 20.04 for arm64](https://ubuntu.com/download/raspberry-pi), although it should work with other Debian-based distributions.

## Usage

The inventory provided in `inventory/hosts.ini` is specific to my configuration and will need to be edited or replaced.

```ini
[all]
192.168.4.42
192.168.4.43
192.168.4.44
192.168.4.45

[all:vars]
ansible_user=k8s
ansible_ssh_private_key=~/.ssh/rpi_cluster_rsa
```

Install dependencies from Ansible Galaxy.

```
$ ansible-galaxy install -r requirements.yml
```

Run the playbook to set up packages, services, and storage.

```
$ ansible-playbook main.yml
```

## Customization

### Storage

You can configure mounts in `/etc/fstab` by providing a list of `storage_devices`. For example, this sets up a mount for [Longhorn](https://longhorn.io).

```yaml
# host_vars/192.168.4.44.yml
---
storage_devices:
  - path: /var/lib/longhorn
    src: LABEL=longhorn
    fstype: ext4
    opts: defaults
```

Similarly, NFS exports can be configured by providing a list of `nfs_exports`. The following example adds a mount to `/etc/fstab` and exports it via NFS.

```yaml
---
storage_devices:
  - path: /cluster-data
    src: UUID=169d8a3d-96f0-4d2d-aa75-a8e81003e7cb
    fstype: ext4
    opts: defaults,nosuid,nodev,nofail
nfs_exports:
  - /cluster-data    10.1.1.0/16(rw,sync,root_squash)
```

Storage support is a simple wrapper around the [Ansible mount module](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html) and the [geerlingguy.nfs role](https://github.com/geerlingguy/ansible-role-nfs). See the `host_vars` directory for full details.
