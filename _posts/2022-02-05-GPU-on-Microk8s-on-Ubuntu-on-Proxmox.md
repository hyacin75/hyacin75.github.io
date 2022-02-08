---
layout: post
title: GPU on Microk8s on Ubuntu on Proxmox
---

Well I think the title says it all, doesn't it?

I was trying not just to pass a GPU through Proxmox to a guest,
but then have that guest pass it through to the Kubernetes
nvidia-gpu-operator to make it available in a k8s cluster.

It's been a fun couple days, but it's finally working.

Documenting the steps so I don't have to figure it out all over
again when I re-build the boxes for real lab deployment once
their secondary hard drives and new 14c/28t CPUs arrive ... oh
and RAM... they came with 32GB each so I ordered 64GB to stick
in one and then I'll move it's RAM into the other so they both
have 64GB.  They'll be like 7x NUC10i3FNH's in terms of CPU
power.  They'll have a lot less memory per core than my NUC10s
do (as they have 32GB in each), but they don't even use all their
memory, and 128GB of 2400 RDIMMS AIN'T CHEAP, so 64 GB will just
have to do.

So anyway -

  Install Proxmox hypervisor
  
  Instruct hypervisor to not use CUDA GPU -
  
  root@t5810-01-t4:~# cat /etc/modprobe.d/blacklist.conf
  blacklist nouveau
  blacklist nvidia
  root@t5810-01-t4:~# cat /etc/modprobe.d/vfio.conf
  options vfio-pci ids=10de:1430,10de:0fba disable_vga=1
  root@t5810-01-t4:~# lspci | grep -i quadro
  04:00.0 VGA compatible controller: NVIDIA Corporation GM206GL [Quadro M2000] (rev a1)
  root@t5810-01-t4:~# lspci -n -s 04:00
  04:00.0 0300: 10de:1430 (rev a1)
  04:00.1 0403: 10de:0fba (rev a1)
  root@t5810-01-t4:~#
  
  reboot.
  
  Install Ubuntu 20.04 guest with the following -
   
  8-10 GB memory MINIMUM
  machine type q35
  SeaBIOS
  At least 64GB usable disk space
  pass through the PCI device from the host with -
  All Functions
  ROM-Bar
  PCI-Express
  all checked (Primary GPU UNchecked)
  
  Blacklist nouveau in grub (no idea why the guest needs it via grub but the /etc/modprobe.d files for fine for the host) -
  
  GRUB_CMDLINE_LINUX_DEFAULT="maybe-ubiquity modprobe.blacklist=nouveau nouveau.modeset=0" (second and third entires are the relevant ones)
  
  reboot.
  
  Install mk8s -
  
  sudo snap install microk8s --classic --channel=1.23
  
  sudo usermod -a -G microk8s $USER
  
  sudo usermod -a -G microk8s $USER
  
  log out/in
      
  enable GPU -
  
  microk8s enable gpu
  
  wait patiently for the operator to finish.


The pod to watch while it's going is nvidia-driver-daemonset -

  microk8s kubectl logs -f pod/nvidia-driver-daemonset-tds7g -n gpu-operator-resources

When that completes, the pods that were stuck in Init should
start to transition through their stages and to Complete.

If they do not, something may have gone awry.

Once they're all done you can do a quick test with the
sample pod from the mk8s docs -

  apiVersion: v1
  kind: Pod
  metadata:
    name: cuda-vector-add
  spec:
    restartPolicy: OnFailure
    containers:
      - name: cuda-vector-add
        image: "k8s.gcr.io/cuda-vector-add:v0.1"
        resources:
          limits:
            nvidia.com/gpu: 1

It's a large container download, but once it finishes that
it runs very quickly and should look like -

  rob@mk8s-gpu-test-05:~$ kubectl logs -f pod/cuda-vector-add
  [Vector addition of 50000 elements]
  Copy input data from the host memory to the CUDA device
  CUDA kernel launch with 196 blocks of 256 threads
  Copy output data from the CUDA device to the host memory
  Test PASSED
  Done
  rob@mk8s-gpu-test-05:~$


And you're done!  Now to find things to use those CUDA cores on!
