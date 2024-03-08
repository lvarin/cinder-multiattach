# Cinder multiattach demo

This is a simple ansible playbook to configure a cluster of VMs for seeing how cinder multi-attach works.

I used this article as the main source: https://docs.syseleven.de/syseleven-stack/en/tutorials/cinder-multiattach

![Architecture](multi-attach.drawio.svg)

The Archtecture is very simple. A single cinder volume, of the type `multiattach`, mounted in everyone of the VM nodes (limited to 255 nodes). Then a small gateway VM to access the VMs by SSH.


