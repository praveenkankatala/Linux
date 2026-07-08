# 6. Cheat Sheet & Practice Quiz

[⬅ Previous: Containers vs VMs](./05-containers-vs-virtual-machines.md) | [🏠 Index](./README.md)

---

## 🔹 Command Cheat Sheet

### Check virtualization support on Linux
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo     # >0 means CPU supports virtualization
lscpu | grep Virtualization             # Shows VT-x / AMD-V support
kvm-ok                                  # (Ubuntu) Confirms if KVM can run
```

### VirtualBox — VBoxManage
```bash
VBoxManage list vms                                    # List all VMs
VBoxManage list runningvms                              # List running VMs
VBoxManage startvm "VM_NAME" --type headless             # Start without GUI
VBoxManage controlvm "VM_NAME" poweroff                  # Force stop
VBoxManage controlvm "VM_NAME" acpipowerbutton            # Graceful shutdown
VBoxManage snapshot "VM_NAME" take "SNAP_NAME"            # Take snapshot
VBoxManage snapshot "VM_NAME" restore "SNAP_NAME"         # Restore snapshot
VBoxManage modifyvm "VM_NAME" --memory 4096                # Change RAM (MB)
VBoxManage showvminfo "VM_NAME"                            # Show VM details
```

### KVM / libvirt (Linux native virtualization)
```bash
virsh list --all                       # List all VMs (running + stopped)
virsh start VM_NAME                    # Start a VM
virsh shutdown VM_NAME                 # Graceful shutdown
virsh destroy VM_NAME                  # Force stop
virsh snapshot-create-as VM_NAME snap1 # Create snapshot
```

### AWS CLI — EC2 essentials
```bash
aws ec2 describe-instances                                # List all instances
aws ec2 start-instances --instance-ids i-xxxxxxxx           # Start instance
aws ec2 stop-instances --instance-ids i-xxxxxxxx             # Stop instance
aws ec2 terminate-instances --instance-ids i-xxxxxxxx          # Permanently delete
aws ec2 describe-security-groups                                # List firewall rules
```

---

## 🔹 Terminology Quick Reference

| Term | One-line meaning |
|------|--------------------|
| Hypervisor | Software that creates & manages VMs |
| Host | Physical machine providing real hardware |
| Guest | OS running inside a VM |
| Type 1 | Bare-metal hypervisor (production) |
| Type 2 | Hosted hypervisor (learning/testing) |
| KVM | Linux's built-in Type 1 hypervisor |
| QEMU | Hardware emulator used with KVM |
| AMI | AWS's VM template |
| EC2 Instance | AWS's term for a running VM |
| Snapshot | Saved state of a VM for instant rollback |
| Container | Lightweight, kernel-sharing isolated app environment |

---

## 🔹 Practice Quiz

Try answering before revealing each answer.

<details>
<summary><b>Q1.</b> What's the main difference between Type 1 and Type 2 hypervisors?</summary>
Type 1 runs directly on physical hardware (bare-metal); Type 2 runs as an application on top of an existing host operating system.
</details>

<details>
<summary><b>Q2.</b> Name three benefits of virtualization.</summary>
Any three of: cost savings, better hardware utilization, isolation, scalability, fast disaster recovery, safe testing, easy migration.
</details>

<details>
<summary><b>Q3.</b> In VirtualBox, which network mode would you use to let two VMs talk to each other but stay completely isolated from the internet?</summary>
Host-Only Adapter (or Internal Network if you also want to exclude the host itself).
</details>

<details>
<summary><b>Q4.</b> What is a VM snapshot used for?</summary>
Saving the exact state of a VM so you can instantly roll back to it later, e.g. before making risky changes.
</details>

<details>
<summary><b>Q5.</b> What replaced much of AWS's traditional software hypervisor to boost performance?</summary>
The AWS Nitro System, which offloads networking/storage/security to dedicated hardware.
</details>

<details>
<summary><b>Q6.</b> What is the cloud (AWS) equivalent of a VirtualBox .iso + saved VM configuration?</summary>
An AMI (Amazon Machine Image).
</details>

<details>
<summary><b>Q7.</b> Why do containers start in seconds while VMs take minutes?</summary>
Containers share the host's existing kernel and only start the application process, while a VM must boot an entire separate operating system from scratch.
</details>

<details>
<summary><b>Q8.</b> What three components make up the core of Linux's KVM-based virtualization stack?</summary>
KVM (kernel module), QEMU (hardware emulation), and libvirt (management layer).
</details>

---

## 🔹 Suggested Practice Projects

1. **Build a 3-tier lab in VirtualBox** — one VM as a web server, one as an app server, one as a database, connected via a Host-Only network
2. **Launch an EC2 instance and install Nginx** — connect via SSH, install a web server, and view the default page from your browser using the instance's public IP
3. **Take and restore a snapshot** — install something in a VM, snapshot it, break something on purpose, then restore and confirm it's fixed
4. **Compare boot times** — time how long a VirtualBox Ubuntu VM takes to boot vs. how long a Docker container running Ubuntu takes to start, and explain why they differ

---

## 🎉 You've completed the Linux Virtualization guide!

Go back to the [Index](./README.md) to review any section, or move on to hands-on labs of your own.

[⬅ Previous: Containers vs VMs](./05-containers-vs-virtual-machines.md) | [🏠 Index](./README.md)
