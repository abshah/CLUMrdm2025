# Data Management in the de.NBI cloud Part 1

This assumes that you are familiar with the de.NBI cloud platform and have setup your account and ssh keys.
We also assume that you are familiar with basic operations such as launching VMs,volumes and containers. However, we will briefly also cover them.

## Section 1: Create a new VM

First, we will start out by creating a new VM via the SimpleVM webportal (https://simplevm.denbi.de/portal/webapp). Select "Create new Instance" and choose the "CLUMrdm20251" project, with "de.NBI small" flavor and Guacamole as your research environment. We will also install some additional Conda Packages. We will install parallel (version 20250822), sra-tools (version 3.2.1) and minio-client (version 2025.08.13.08.35.41) conda packages.

Next, grant access to the workshop organizers (**Sebastian Jünemann** and **Abhijeet Shah**). This way these memebers get ssh access to your VM in case something does not work as expected. 

Once the VM is built and spooled up (it may take upto 10 minutes), you can connect to it via SSH or Guacamole. Launch a terminal (if you use Guacamole), and fetch the latest information about available packages and updates. Following this, we will also install a few packages.

```bash
sudo apt update
sudo apt upgrade
sudo apt install python3-openstackclient
```
## Section 2: Create a volume to store data

1. Inspect what block storage is available on your virtual instance by typing:

   ``` bash
   df -h 
   ```

2. Check the current disk usage and available space.

3. In the SimpleVM portal, navigate to the Volumes section and create a volume for your VM.
   Enter your name without whitespace (Example: Max Mustermann -> MaxMusterman) as the volume name 
   and provide 10 GB as the volume size.
   ![Create Volume](figures/createVolume.png)

4. Attach this volume to your instance by opening the pull down menu of your volume and
   clicking the green attach icon. Then select your virtual machine in the pop up menu and
   click the attach button.
   ![Attach Volume](figures/attachVolume.png)

5. Activate the newly created and attached volume in your VM:

   ``` bash
   lsblk
   sudo mkfs.ext4 /dev/vdc
   sudo mkdir /mnt/volume   
   sudo mount /dev/vdc /mnt/volume
   sudo chown ubuntu:ubuntu /mnt/volume
   lsblk   
   ```
 
![lsblk](figures/lsblk.png)

![mkfs_ext4](figures/mkfs_ext4.png)

## Section 3: Using Object Storage

In this section, we will the openstack horizon interface directly.
First we need to setup and configure our credentials on our virtual machine.
Please navigate to the following site:

<https://openstack.cebitec.uni-bielefeld.de/project/>

1. Creating application credentials

In this section you will create an [application
credential](https://docs.redhat.com/en/documentation/red_hat_openstack_platform/14/html/users_and_identity_management_guide/application_credentials)
and download the auto-generated `clouds.yaml`. `clouds.yaml` contains all
required authentication information. Follow the images:

![Navigation](images/ac_screen1.png)

Don't use the input field secret. As you can see its input is not
hidden. OpenStack will generate a strong secret for you, if you leave it
blank. You should pick a sensible expiration date.

![Creation](images/ac_screen2.png)

We will now save the downloaded `clouds.yaml` under
`~/.config/openstack/`. That will allow the `OpenstackClient` to access
it. You will also need the `app-cred-openrc.sh` script finish the setup.

![Download](images/ac_screen3.png)

If the file was auto downloaded, you need to move it instead:

``` bash
mkdir -p ~/.config/openstack
mv ~/Downloads/clouds.yaml ~/.config/openstack/
mv ~/Downloads/app-cred*.sh ~/.config/openstack/
chmod 600 ~/.config/openstack/clouds.yaml
chmod 600 ~/.config/openstack/app-cred*.sh
source ~/.config/openstack/app-cred*.sh
```

If you have `OpenstackClient` installed and `openstack subnet list` runs
without error, you are ready to proceed.

``` bash
openstack project list
openstack project show clumRDM251
```

2. Creating S3 credentials

The creation of credentials for the project related object storage can't
be done in the web interface. Therefore, we will use the openstack CLI
for that.

``` bash
openstack --os-identity-api-version 3 ec2 credentials create
```

This command will return you the newly generated key and secret. You
can, at any time, look up what S3 credentials are still valid for you
using

``` bash
openstack --os-identity-api-version 3 ec2 credentials list
```

We will now configure the S3 minio client:

``` bash
mc alias set clumRDM251 https://openstack.cebitec.uni-bielefeld.de:8080/ <YOUR-ACCESS-KEY> <YOUR-SECRET-KEY>
```

3. Uploading data to the Object Storage

We will now use the minio client to upload some data. In the guacamole
SimpleVM instance, type:

``` bash
cd /mnt/volume
```

We will download some more data to play around with:
**Attention**: You will download a large file (approx 14 GiB)

``` bash
mc cp --recursive aws/sra-pub-run-odp/sra/ERR14244221 ./
```

4. Using a object storage container
Next, we are going to create a new object storage container, a so called
bucket. For this, we will use for the first time the horizon interface.
Navigate to the **Container** entry under the **Object Store** menu
entry. Note: All containers here are visible to all project members, as
those containers in openstack are bound to the project

This container is empty, but we can show it nevertheless on the command
line using the minio client:

``` bash
mc ls clumRDM251
```

This should show you your previously created bucket (container) name
(next to all others). You can now upload data into it.

``` bash
mc cp *.fastq.gz clumRDM251/YOUR_CONTAINER_NAME
mc ls clumRDM251/YOUR_CONTAINER_NAME
```

Tip: You can enable auto completion for the minio client. After
activiation, the shell needs to be restarted, though.

``` bash
mc --autocompletion
```

## Section 4: Transferring your data into the cloud

As expected, you may want to upload or transfer data from your local (or remote) machine into a SimpleVM instance or any other virtual machine, then `scp` and `rsync` are your friends.

1. Using `scp` to transfer file(s).

`scp` stands for "secure copy." It is a simple command-line tool that lets you copy files or folders from your own computer to a virtual machine (like SimpleVM), or the other way around, over the network or locally to another user. It works a lot like the `cp` command, but it can transfer files between different computers securely. However, this tool is cumbersome to use to transfer many files or large datasets.

The basic context of `scp` goes somewhat like this

```
scp [option(s)] source destination
```

However, this has to be modified and adapted to our systems. More specifically, like this:

```bash
scp -P Your_port -i /path/to/your/identity_file /path/to/your/source_file user@destination_machine:/path/to/your/destination_file
```



2. Using rsync to transfer many files
`rsync` is a powerful and efficient tool for copying and synchronizing files and directories between your local machine and a remote server (like your SimpleVM), or between two remote servers. It is especially useful when you want to transfer large amounts of data or keep directories in sync, because it only transfers the differences between source and destination. It also provides advanced features such as resuming interrupted transfers and just copying the files that have changed. It can also check for integratity to tranferred files, thereby minimizing the risk of data loss due to corruption. In the IT domain, it is also commonly used for unsupervised and regular automated backups at scale.

The command syntax of `rsync` is similar to `scp`. You can also pass extra `ssh` options to `rsync` (see below for examples).

```bash
rsync [OPTIONS] source_folder_or_file destination_folder_or_file 
```
For example to copy entire directories between instances, enable compression to save network bandwidth, and exclude specific file types.

```bash
# Sync directories between instances
rsync -avz -e "ssh -p YOUR_PORT" ubuntu@YOUR_VM_IP:/mnt/volume/data/  ubuntu@YOUR_OTHER_VM_IP:/PATH/TO/data

# Transfer with compression and progress
rsync -avzP -e "ssh -p YOUR_PORT" ~/large_dataset/ ubuntu@YOUR_VM_IP:/mnt/volume/

# Exclude certain file types.
rsync -avz --exclude="*.tmp" --exclude="*.log" -e "ssh -p YOUR_PORT" ~/data/ ubuntu@YOUR_VM_IP:/mnt/volume/

# Maximize compression during transfer to reduce network costs and load.
rsync -avz --compress-level=9 -e "ssh -p YOUR_PORT" ~/data/ ubuntu@YOUR_VM_IP:/mnt/volume/
```



