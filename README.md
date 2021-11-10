# OpenShift 3.11 on Libvirt

This repository will deploy Red Hat OpenShift 3.11 on Libvirt/KVM hosts.

## Prerequisites

- Get a [Red Hat API Offline Token](https://access.redhat.com/management/api) - this is used to automatically download the RHEL 7 ISO, but can be skipped with the `download_rhel` tag
- If using OSEnterprise, you'll need some RH Registry credentials: https://access.redhat.com/RegistryAuthentication
- Install the Ansible Collections needed: `ansible-galaxy collection install -r requirements.yml`
- Install Python + PIP3 + netaddr: `dnf install python3-pip -y && pip3 install netaddr`

## Deployment

1. Copy `example_vars/cluster-config.yaml` to the working directory, ideally with a prefix of the cluster name - modify as needed

```bash
cp example_vars/cluster-config.yaml my-cluster.cluster-config.yaml
```

2. With the needed variables altered, you can run the Playbook with the following command:

```bash
ansible-playbook --forks 10 -e "@my-cluster.cluster-config.yaml" bootstrap.yaml
```