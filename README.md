# oVirt custom fencing agent for libvirtd-based physical hypervisors

This helps testing oVirt power management (i.e. fencing) features while running
nested under physical hypervisor running libvirtd.

**This is obviously very insecure, only use in a controlled lab setting.**

## Deployment

### oVirt engine VM

```sh
engine-config -s CustomVdsFenceType="libvirt"
engine-config -s CustomVdsFenceOptionMapping="libvirt"

systemctl restart ovirt-provider-ovn ovirt-engine
```

### oVirt cluster hosts

`/usr/sbin/fence_libvirt` should be present on each cluster member.

```yml
    - name: check if vdsm user exists
      ansible.builtin.command:
        cmd: id -u vdsm

    - name: install virsh fence agent
      copy:
        src: "{{ src_dir + item }}"
        dest: /usr/sbin
        owner: root
        group: root
        mode: 0755
      with_items:
        - /usr/sbin/fence_libvirt

    - name: ensure ~vdsm/known_hosts exists
      file:
        path: /var/lib/vdsm/.ssh/known_hosts
        owner: vdsm
        group: kvm
        state: present

    - name: ensure ~vdsm/.ssh exists
      file:
        path: /var/lib/vdsm/.ssh
        state: directory
        mode: 0700
        owner: vdsm
        group: kvm

    - name: install vdsm private key used to access libvirtd hypervisor
      copy:
        dest: /var/lib/vdsm/.ssh/id_ed25519
        mode: 0600
        owner: vdsm
        group: kvm
        content: |
          -----BEGIN OPENSSH PRIVATE KEY-----
          .....
          -----END OPENSSH PRIVATE KEY-----
```