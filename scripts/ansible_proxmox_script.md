# Ansible script for Virtual machines set-up
- hosts: pmx
  tasks:
    - name: Create Juju Controller
      community.general.proxmox_kvm:
        api_host: 10.6.0.1
        api_user: root@pam
        api_password: ***************
        node: pmx
        vmid: 501
        name: JUJU-controller
        tags: mngmnt
        cpu: host
        cores: 2
        memory: 4096
        virtio: {"virtio0":"zfs0:41,discard=on,iothread=1,format=raw"}
        scsihw: virtio-scsi-pci
        net: {"net0":"virtio=BC:24:11:AB:A6:01,bridge=vmbr0,firewall=0","net1":"virtio=BC:24:11:AB:A6:21,bridge=vmbr0,firewall=0"}
        boot: "order=net0;virtio0"
        state: present
      - name: Create Node1
      community.general.proxmox_kvm:
        api_host: 10.6.0.1
        api_user: root@pam
        api_password: ***************
        node: pmx
        vmid: 510
        name: node1
        tags: compute
        cpu: host
        cores: 16
        memory: 71680
        virtio: {"virtio0":"zfs0:96,discard=on,iothread=1,format=raw", "virtio1":"zfs0:128,discard=on,iothread=1,format=raw","virtio2":"zfs0:128,discard=on,iothread=1,format=raw","virtio3":"zfs0:128,discard=on,iothread=1,format=raw"}
        scsihw: virtio-scsi-single
        net: {"net0":"virtio=BC:24:11:AB:A6:10,bridge=vmbr0,firewall=0","net1":"virtio=BC:24:11:AB:A6:20,bridge=vmbr0,firewall=0"}
        boot: "order=net0;virtio0"
        state: present
    - name: Create Node2
      community.general.proxmox_kvm:
        api_host: 10.6.0.1
        api_user: root@pam
        api_password: ***************
        node: pmx
        vmid: 511
        name: node2
        tags: compute
        cpu: host
        cores: 16
        memory: 71680
        storage: "disk0"
        virtio: {"virtio0":"zfs0:96,discard=on,iothread=1,format=raw", "virtio1":"zfs0:128,discard=on,iothread=1,format=raw","virtio2":"zfs0:128,discard=on,iothread=1,format=raw","virtio3":"zfs0:128,discard=on,iothread=1,format=raw"}
        scsihw: virtio-scsi-single
        net: {"net0":"virtio=BC:24:11:AB:A6:11,bridge=vmbr0,firewall=0","net1":"virtio=BC:24:11:AB:A6:41,bridge=vmbr0,firewall=0"}
        boot: "order=net0;virtio0"
        state: present
    - name: Create Node3
      community.general.proxmox_kvm:
        api_host: 10.6.0.1
        api_user: root@pam
        api_password: ***************
        node: pmx
        vmid: 512
        name: node3
        tags: compute
        cpu: host
        cores: 16
        memory: 71680
        storage: "disk0"
        virtio: {"virtio0":"zfs0:96,discard=on,iothread=1,format=raw", "virtio1":"zfs0:128,discard=on,iothread=1,format=raw","virtio2":"zfs0:128,discard=on,iothread=1,format=raw","virtio3":"zfs0:128,discard=on,iothread=1,format=raw"}
        scsihw: virtio-scsi-single
        net: {"net0":"virtio=BC:24:11:AB:A6:12,bridge=vmbr0,firewall=0","net1":"virtio=BC:24:11:AB:A6:22,bridge=vmbr0,firewall=0"}
        boot: "order=net0;virtio0"
        state: present

      
