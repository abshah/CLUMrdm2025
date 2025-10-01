# Data Management in the de.NBI cloud Part 2

## Using s3fs-fuse to mount a S3 container to a virtual machine

1. Installing s3fs in your virtual machine.

Installing s3fs-fuse on SimpleVM is:

```bash
sudo apt update
sudo apt upgrade
sudo apt install s3fs
```

2. Prepare credentials

List your openstack credentials as we did previously using

```bash
openstack --os-identity-api-version 3 ec2 credentials list
```

If you encounter an error, you may need to re-run the script we downloaded from the Horizon Openstack interface.

```bash
source ~/.config/openstack/*.sh
```

Use the listed access key and secret from our workshop project. If you are part of multiple simpleVM projects, be sure to pick the correct one.


```bash
echo ACCESS_KEY_ID:SECRET_ACCESS_KEY > ${HOME}/.passwd-s3fs
```

Modify the permissions of the credentials file with appropriate file permissions.


```bash
chmod 600 ${HOME}/.passwd-s3fs
```

Allow non-root users to read and write to the mount.

```bash
sudo sed -i 's/^#user_allow_other/user_allow_other/' /etc/fuse.conf
```


3. Create the mount point.

Similar to mounting a volume, we will also create a new directory and set permissions.

```bash
sudo mkdir -p /mnt/s3
sudo chown ubuntu:ubuntu /mnt/s3
```

4. Test a foreground mount

We shall run s3fs in an interactive environment to see if it can run without errors. 

```bash
s3fs YOUR_CONTAINER /mnt/s3 \
  -o passwd_file=${HOME}/.passwd-s3fs \
  -o url=https://openstack.cebitec.uni-bielefeld.de:8080 \
  -o use_path_request_style \
  -o allow_other \
  -o uid=$(id -u) -o gid=$(id -g) \
  -o umask=0022
```

5. Verify the mount

Write a small file to confirm permissions and configuration. You will need to do so from another terminal as your current terminal is running s3fs.

```bash
echo "hello" > /mnt/s3/s3fs_test.txt
cat /mnt/s3/s3fs_test.txt
```

If you can run these commands successfully, then we have succeeded in mounting the container.

6. Configure automatic mounting

Copy your credentials to a location appropriate for the root user. 

```bash
sudo cat ${HOME}/.passwd-s3fs > /etc/passwd-s3fs
sudo chmod 600 /etc/passwd-s3fs
```

Add a new entry to your `/etc/fstab` file. 

```bash
MY_CONTAINER /mnt/s3 fuse.s3fs _netdev,allow_other,use_path_request_style,url=https://openstack.cebitec.uni-bielefeld.de:8080,passwd_file=/etc/passwd-s3fs,uid=1000,gid=1000,umask=0022,x-systemd.requires=network-online.target,x-systemd.automount 0 0
```

Apply changes without rebooting

```bash
sudo systemctl daemon-reload
sudo systemctl restart systemd-networkd || true
sudo systemctl start network-online.target
sudo mount -a
```

