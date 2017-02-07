# dti_machine

## Notes on setting up a DTI machine on AWS

#### Step 1: Create an AWS EC2 Instance

You need to go to the AWS Console -> EC2 -> Launch Instance

Choose Ubuntu 14.04 LTS as the Operating System

Choose the g2.2xlarge Instance Type

_If you haven't created a keypair yet, now is a good time to do so_

Assign a key pair and launch the instance.

Before you do anything else, go to the AWS Console -> EC2 -> Instances and select your new instance; go to 
Actions -> Instance Settings -> Change Shutdown Behavior and make sure that 'Stop' is selected.

#### Step 2: SSH into your EC2 Instance

Go to your AWS Console -> EC2 -> Instances

Find the machine you just launched. _Now is a good time to name the instance_.

Select the Instance, and click "Connect". You'll get a pop-up with detailed connection instructions.

Open a terminal and type the SSH command to connect. e.g.:

`ssh -i "keypair.pem" ubuntu@ec2-01-001-1-01.computer-1.amazonaws.com`

#### Step 3: Prepare the OS

Check for updates to pre-installed packages
`sudo apt-get update`

Install tools for compiling new drivers/software
```
sudo apt-get install build-essential
sudo apt-get install linux-image-extra-virtual
```

Get CUDA Installer
`wget http://developer.download.nvidia.com/compute/cuda/6_5/rel/installers/cuda_6.5.14_linux_64.run`

Extract Installers
```{bash}
chmod +x cuda_6.5.14_linux_64.run
mkdir nvidia_installers
./cuda_6.5.14_linux_64.run -extract=`pwd`/nvidia_installers
```

Disable _nouveau_

sudo open a new file at this path: `/etc/modprobe.d/blacklist-nouveau.conf`

and add these lines to it:
```
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```

Then disable the kernel nouveau

`echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf`

You should see `options nouveau modeset=0`

Reboot the system
```
sudo update-initramfs -u
sudo reboot
```

Install the Kernel source
```
sudo apt-get install linux-source
sudo apt-get install linux-headers-`uname -r`
```

#### Step 4: Install NVIDIA drivers

```{bash}
cd nvidia_installers
sudo ./NVIDIA-Linux-x86_64-340.29.run
```

Load the nvidia kernel module

`sudo modprobe nvidia`

Run CUDA + samples installer
```
sudo ./cuda-linux64-rel-6.5.14-18749181.run
sudo ./cuda-samples-linux-6.5.14-18745345.run
```

Verify CUDA is correctly installed
```
cd /usr/local/cuda/samples/1_Utilities/deviceQuery
make
./deviceQuery   
```
You should see the following output:
```
deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 6.5, CUDA Runtime Version = 6.5, NumDevs = 1, Device0 = GRID K520
Result = PASS
```

Step 5: Install FSL from source

Step 6: Install Dropbox (optional)






## References:

#### For installing Dropbox

#### For installing FSL from source

#### For installing CUDA
http://tleyden.github.io/blog/2014/10/25/cuda-6-dot-5-on-aws-gpu-instance-running-ubuntu-14-dot-04/
