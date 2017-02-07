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

Run CUDA installer
```
sudo ./cuda-linux64-rel-6.5.14-18749181.run
```
Optional: Run samples installer
```
sudo ./cuda-samples-linux-6.5.14-18745345.run
```

Set up the environment paths for CUDA

Open your `~/.bash_profile` file and add the following lines:
```{bash}
 CUDA_HOME=/usr/local/cuda-6.5
 LD_LIBRARY_PATH=${CUDA_HOME}/lib64
 export LD_LIBRARY_PATH
 
 PATH=${CUDA_HOME}/bin:${PATH}
 export PATH
```

To include these environment variables in your current environment, run `source ~/.bash_profile`

Verify CUDA is correctly installed (Only if Optional samples files were installed. See above.)
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

#### Step 5: Install FSL from source

Enter the following to download the fsl source code to the machine
```
curl --header 'Host: fsl.fmrib.ox.ac.uk' --header 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:50.0) Gecko/20100101 Firefox/50.0' --header 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' --header 'Accept-Language: en-US,en;q=0.5' --header 'Referer: https://fsl.fmrib.ox.ac.uk/fsldownloads/fsldownloadmain.html' --header 'Connection: keep-alive' --header 'Upgrade-Insecure-Requests: 1' 'https://fsl.fmrib.ox.ac.uk/fsldownloads/fsl-5.0.9-sources.tar.gz' -o 'fsl-5.0.9-sources.tar.gz' -L
```

#### Step 6: Expand the space on your machine

To expand the space on your machine (A likely necessity because the default attached storage limit is 8GB),
you need to first shutdown the machine. _Before you to this, please ensure that you've changed the shutdown behavior
as in Step 1.

Take note of the "Availability Zone" for the machine.

Once the machine is stopped. Go to the AWS Console -> EC2 -> Volumes

Find the volume attached to your instance (use the Created date field for help) and select it

Rename it to `dti-machine-small` to make it easier to identify later

Go to Actions -> Detach Volume

Once the volume has been detached, select it again and go to Actions -> Create Snapshot

Name the snapshot `dti-machine-small` and click Create

Go to AWS Console -> EC2 -> Snapshots

The snapshot make take some time.

Once the `dti-machine-small` snapshot is ready, select it and go to Actions -> Create Volume

Select the size you desire, and the Availability Zone you noted earlier from the machine, and
press Create

Go back to AWS Console -> EC2 -> Volumes

Once the volume is created, select it and go to Actions -> Attach Volume

Click in the Instance field and select your Instance, select `/dev/sda1` for the Device and press Attach

Go back to AWS Console -> EC2 -> Instances and restart your instance (select and go to Actions -> Instance State -> Start)

SSH back into your machine (See Step 2 above; note the IP address may have changed).

Type `df -h` on the Console to see the free space (look at the /dev/xvda1 line).

Step 7: Install Dropbox (optional)




## References:

#### For installing Dropbox

#### For installing FSL from source

#### For installing CUDA
http://tleyden.github.io/blog/2014/10/25/cuda-6-dot-5-on-aws-gpu-instance-running-ubuntu-14-dot-04/

#### Storage expansion
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-expand-volume.html
