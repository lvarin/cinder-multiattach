# Cinder multiattach demo

This is a simple ansible playbook to configure a cluster of VMs for seeing how cinder multi-attach works. It is more a practical documentation than a fully fledged playbook. It can be configured using two different filesystems: `ocfs2` and `gfs2`. For the `ocfs2` filesystem, this article has been used as the main source:

<https://docs.syseleven.de/syseleven-stack/en/tutorials/cinder-multiattach>

![Architecture](multi-attach.drawio.svg)

The Architecture is very simple. A single cinder volume, of the type `multiattach`, mounted in every single one of the VM nodes (limited to 255 nodes in `ocfs2`). Each node runs a daemon that has configured the IP or DNS names of the other nodes and use that connection to coordinate file system tasks (read, write and so), and administrative tasks.

A small jump host VM needs to be present in order to access the VMs by SSH.

# Deploy

The prerequisites are installed like this:

```sh
$ pip install -r requirements.txt

$ ansible-galaxy install -r requirements.yml 

$ source <openrc.sh file>
Please enter your OpenStack Password for project XXXXXXXXXXX as user YYYYYYYY:
```

Then to create the VMs and install the default file system `ocfs2`, call ansible:

```sh
$ ansible-playbook main.yml
```

If you want to try `gfs2` do:

```sh
ansible-playbook main.yml -e fs='gfs2'
```

ocfs2 will be installed on Ubuntu 22.04, and gfs2 in Almalinux 9. Defaults work out of the box, but you can edit `group_vars/all.yml` to change them.

# Bugs

1. The ansible module does not seem to allow multimount, this might be solved with a newer version of the module or OpenStack SDK, but for the time being, one needs to do the mount manually after Ansible has created the volume:

  ```sh
  openstack --os-compute-api-version 2.60 server add volume "<SERVER>" <VOLUME>
  ```

  ...for each of the severs. One way to make it less manual is to do:

  ```sh
  for i in $(seq 1 3);
  do
    openstack --os-compute-api-version 2.60 server add volume "cinder-$i" multi-attach-test
  done
  ```

This assumes that there are `3` servers, the prefix is `cinder` and the volume is called `multi-attach-test`. All of this is configured in the `group_vears/all.yml` file.

2. The Format of the partition is not working well. More tests needed. But one can do the creation manually:

  * For `ocfs2`:

    ```sh
    sudo mkfs.ocfs2 /dev/vdb -N 16
    ```

    `16` is the maximum number of nodes and the default. So only need to use `-N` if we need more clients than that. Ansible now runs with `-N <number_instances>`.

  * For `gfs2`:
    ```sh
    sudo mkfs.gfs2 -p lock_dlm -t gfs_cluster:mygfs2 -j 4 /dev/vdb
    ```

    `4` is size of the journal, thus the maximum number of nodes.

**NOTE**: This is only tested in our OpenStack environment, for the current available software. YMMV
