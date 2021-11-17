### **vfio-single-gpu-passthrough-guide**
Personal guide for single GPU passhtrough with KVM on Manjaro derived from [Risingprism guide](https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/home) and [QaidVoid guide](https://github.com/QaidVoid/Complete-Single-GPU-Passthrough)

## **Table of contents**
* **Personal Setup**
* **IOMMU Setup**
* **Installing and configuration of Libvirt**
* **Guest Setup**
* **Prepare and attach PCI devices**
* **Libvirt Hooks**
* **Todos**

## **Personal Setup**
### **Hardware Specs**

### **BIOS Settings**
Enable ***Intel VT-d*** or ***AMD-Vi*** virtualization in BIOS. Hardware needs to support one of these virtualization options, otherwise this can not work. 

## **IOMMU Setup**
### **GRUB**
Enable IOMMU support to grub.

| /etc/default/grub |
| ----- |
| `GRUB_CMDLINE_LINUX_DEFAULT="... intel_iommu=on iommu=pt ..."` |
| OR |
| `GRUB_CMDLINE_LINUX_DEFAULT="... amd_iommu=on iommu=pt ..."` |

Update grub and reboot system.
```sh
sudo update-grub
```
Verify if IOMMU is enabled.
```sh
dmesg | grep 'IOMMU enabled'
```

### **Identify PCI devices** 
Use this script to list and group the IOMMU groups
```sh
#!/bin/bash
shopt -s nullglob
for g in /sys/kernel/iommu_groups/*; do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```
or just use this command for listing all PCI devices
```sh
lspci 
```
or filter for AMD or NVIDIA specific devices
```sh
lspci | grep AMD|NVIDIA
```
However it is the best to use the script to identify if the graphics card is isolated in a own IOMMU group. (Does the GPU really needs to be in an own IOMMU group to work?). Find the ID of the GPU and the associated audio controller and note them somewhere.

## **Installing and configuration of Libvirt**
Virtualization of guest system will be done with libvirt and qemu which have to be installed first. \
**Arch Linux/ Manjaro**
```sh
sudo pacman -S virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat ovmf
```
The config of libvirtd has to be changed to grant access rights for libvirt.
```sh
sudo nano /etc/libvirt/libvirtd.conf
```

Search and uncomment the following lines:
* ```unix_sock_group = "libvirt```
* ```unix_sock_rw_perms = "0770```

For logs append the follwing lines at the end:
* ```log_filters="1:qemu"```
* ```log_outputs="1:file:/var/log/libvirt/libvirtd.log```

The config for qemu has to be changed to set access rights.
```sh
sudo nano /etc/libvirt/qemu.conf
```

Search and change the following lines:
* ```#user = yourUsername```
* ```#group = yourGroup```

Add libvirt user to sudo group.
```sh
sudo usermod -a -G libvirt $(whoami)
```

Start and enable libvirtd.
```sh
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
```

## **Guest Setup**

## **Prepare and attach PCI devices**

## **Libvirt Hooks**

## **Todos**
- [ ] CPU pinning
- [ ] Passthrough a complete drive into VM instead of using a VirtIO drive (better disk performance?)
- [ ] Improve startup.sh and teardown.sh for other dipsplaymanager
