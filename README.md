# Cinder multiattach demo

This is a simple ansible playbook to configure a cluster of VMs for seeing how cinder multi-attach works.

I used this article as the main source: https://docs.syseleven.de/syseleven-stack/en/tutorials/cinder-multiattach

![Architecture](multi-attach.drawio.svg)

The Archtecture is very simple. A single cinder volume, of the type `multiattach`, mounted in everyone of the VM nodes (limited to 255 nodes). Then a small gateway VM to access the VMs by SSH.

# Deploy

```sh
$ pip install -r requirements.txt

$ ansible-galaxy install -r requirements.yml 

$ source <openrc.sh file>
Please enter your OpenStack Password for project XXXXXXXXXXX as user YYYYYYYY:

$ ansible-playbook main.yml
```

# Bugs

1. The ansible module does not seem to allow multimount, this might be solved with a newer version of the module or OpenStack SDK, but for the timne being, one needs to do the mount manually after Ansible has created the volume:

  ```sh
  openstack --os-compute-api-version 2.60 server add volume "<SERVER>" <VOLUME>
  ```

  for each of the severs. One way to make it less manual is to do:

  ```sh
  for i in $(seq 1 3);
  do
    openstack --os-compute-api-version 2.60 server add volume "cinder-$i" multi-attach-test
  done
  ```

  This assumes that there are `3` servers, the prefix is `cinder` and the volume is called `multi-attach-test`.

2. The Format of the partition is not working well. More tests needed. But one can do the creation manually:

  ```sh
  sudo mkfs.ocfs2 /dev/vdb -N 16
  ```

  `16` is the maximum number of nodes and the default. So only need to use `-N` if we need more clients than that. Ansible now runs with `-N <number_instances>`.

