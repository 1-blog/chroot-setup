### Step 1: Download the Ubuntu Filesystem (On your Laptop)
```bash 
wget https://cdimage.ubuntu.com/ubuntu-base/releases/noble/release/ubuntu-base-24.04.4-base-arm64.tar.gz
```
(If the link 404s, just navigate to cdimage.ubuntu.com/ubuntu-base/releases/ in your browser and copy the link for the latest ARM64 .tar.gz)

### Step 2: Push and Extract (On the Phone)
 ```bash 
 adb push ubuntu-base-24.04.4-base-arm64.tar.gz /data/local/tmp/
```
[enable root debugging in the developer setting in the phone]
```bash 
adb root
```
```bash 
adb shell
```
```bash 
mkdir -p /data/local/ubuntu
```
```bash 
cd /data/local/ubuntu
```
```bash 
tar -xzf /data/local/tmp/ubuntu-base-24.04.4-base-arm64.tar.gz
```

### Step 3: Create the "Start Script"

(just copy paste it)
```bash
cat << 'EOF' > /data/local/start_ubuntu.sh
#!/system/bin/sh
export MNT=/data/local/ubuntu

# Put SELinux in permissive mode so it doesn't block chroot
setenforce 0

# Mount virtual filesystems if not already mounted
mountpoint -q $MNT/proc || mount -t proc proc $MNT/proc
mountpoint -q $MNT/sys || mount -t sysfs sys $MNT/sys
mountpoint -q $MNT/dev || mount -o bind /dev $MNT/dev
mountpoint -q $MNT/dev/pts || mount -o bind /dev/pts $MNT/dev/pts

# Fix Android DNS mapping
echo "nameserver 8.8.8.8" > $MNT/etc/resolv.conf
echo "127.0.0.1 localhost" > $MNT/etc/hosts

echo "Entering Ubuntu... Type 'exit' to leave."
# Start chroot with proper environment variables
chroot $MNT /usr/bin/env -i \
    HOME=/root \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin \
    TERM=$TERM \
    /bin/bash --login
EOF
```
Make it executable:
```bash
chmod +x /data/local/start_ubuntu.sh
```
start the script:
```bash
/data/local/start_ubuntu.sh
```
[Add the crucial Android networking groups]
```bash
groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
usermod -aG aid_inet root
usermod -aG aid_net_raw root
```

[close the session and reopen and run `apt update` if it fails run below command]
```bash
# 1. Give the hidden _apt user Android internet permissions
usermod -aG aid_inet _apt
# 2. Tell apt not to use the sandbox (forces it to just use root)
echo 'APT::Sandbox::User "root";' > /etc/apt/apt.conf.d/99-chroot-sandbox
# 3. Reload your current shell so the internet groups finally activate
su - root
# 4. Try the update again!
apt update
apt upgrade -y
```
*voila you have an fully functional ubuntu terminal*

### Now we install tmux and python and others to test this ubuntu
tmux helps to run any service in background even if we exit from the adb shell, without for eg: python server stop as soon as we exit ubuntu
```bash
apt install -y python3 wget tmux nano
```
