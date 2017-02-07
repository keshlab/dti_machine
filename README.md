# dti_machine

## Notes on setting up a DTI machine on AWS

Step 1: Create an AWS EC2 Instance

You need to go to the AWS Console -> EC2 -> Launch Instance

Choose Ubuntu 14.04 LTS as the Operating System

Choose the g2.2xlarge Instance Type

_If you haven't created a keypair yet, now is a good time to do so_

Assign a key pair and launch the instance.

Step 2: SSH into your EC2 Instance

Go to your AWS Console -> EC2 -> Instances

Find the machine you just launched. _Now is a good time to name the instance_.

Select the Instance, and click "Connect". You'll get a pop-up with detailed connection instructions.

Open a terminal and type the SSH command to connect. e.g.:

`ssh -i "keypair.pem" ubuntu@ec2-01-001-1-01.computer-1.amazonaws.com`

Step 3: Prepare the OS

Check for updates to pre-installed packages
`sudo apt-get update`

Install tools for compiling new drivers/software
`sudo apt-get install build-essential`

Get CUDA Installer:
`wget http://developer.download.nvidia.com/compute/cuda/6_5/rel/installers/cuda_6.5.14_linux_64.run
`
