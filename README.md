# OpenShift 3.11 on Libvirt

This repository will deploy Red Hat OpenShift 3.11 on Libvirt/KVM hosts.

## Prerequisites

- Get a [Red Hat API Offline Token](https://access.redhat.com/management/api) - this is used to automatically download the RHEL 7 ISO, but can be skipped with the `download_rhel` tag
- If using OSEnterprise, you'll need some RH Registry credentials: https://access.redhat.com/RegistryAuthentication
- Install the Ansible Collections needed: `ansible-galaxy collection install -r requirements.yml`
- Install Python + PIP3 + netaddr: `dnf install python3-pip -y && pip3 install netaddr`

## Architecture

By default the architecture supported by this deployer is:

- 3 Control Plane nodes running co-located etcd and other infrastructure services
- 3 Application Nodes
- A Load Balancer for the API, HTTP, and HTTPS endpoints

It assumes DNS is in place for all the hostnames with `{api,api-int,*.apps}.$CLUSTER_NAME.$CLUSTER_DOMAIN` being set to the Load Balancer IP

## Deployment

1. Copy `example_vars/cluster-config.yaml` to the working directory, ideally with a prefix of the cluster name - modify as needed

```bash
cp example_vars/cluster-config.yaml my-cluster.cluster-config.yaml
```

2. With the needed variables altered, you can run the Playbook with the following command:

```bash
ansible-playbook --forks 10 -e "@my-cluster.cluster-config.yaml" bootstrap.yaml
```

3. Once cluster bootstrapping is complete, you will need to connect to the Load Balancer node and run the actual OSE installation playbooks:

```bash
## Connect to the load balancer with the generated SSH Keys
ssh -i ./.generated/ocp3/id_rsa ocpthree@lb

## Once inside the LB Terminal:
cd /usr/share/ansible/openshift-ansible
ansible-playbook -i /home/ocpthree/ose_hosts playbooks/prerequisites.yml
ansible-playbook -i /home/ocpthree/ose_hosts playbooks/deploy_cluster.yml
```

4. With the cluster deployed, there are still a few steps to make it usable - this is automated by the `configure.yaml` playbook from this repo:

```bash
## Exit the LB node, return to the Ansible control node that has this repo cloned
ansible-playbook --forks 10 -e "@my-cluster.cluster-config.yaml" configure.yaml
```

This playbook will set up HTPasswd auth and reconfigure the Load Balancer to proxy the application routes as well.

## Deleting the cluster

```bash
ansible-playbook --forks 10 -e "@my-cluster.cluster-config.yaml" destroy.yaml
```