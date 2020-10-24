# ebooks
Ebook solution test


# Components

## NAS
* VM size = A1
* Ubuntu 18.04 server

### Configuration script
```
sudo apt update
sudo apt install -y nfs-kernel-server
sudo mkdir -p /exports/ebooks
sudo chown -R nobody:nogroup /exports/ebooks
sudo chmod 777 /exports/ebooks

# Replace ip# by webserver's IP in command below
echo "/exports/ebooks 10.0.0.5(rw,sync,no_subtree_check)" | sudo tee -a /etc/exports

sudo exportfs -a
sudo systemctl restart nfs-kernel-server

```

## WEBDAV enabled webserver
* VM size = A1
* Ubuntu 18.04 server

### Configuration script
```
# Install NFS client
sudo apt update
sudo apt install -y nfs-common
# Replace ip# by nas's IP in command below
echo "10.0.0.4:/exports/ebooks /mnt/ebooks nfs defaults 0 0" | sudo tee -a /etc/fstab
sudo mount -a

```

```
# Install webserver
sudo apt install -y nginx-full
# Optional: set gzip compression
sed -i '/gzip_/ s/#\ //g' /etc/nginx/nginx.conf
echo -n 'testUser:' | sudo tee -a /etc/nginx/.credentials.list
openssl passwd -apr1 testPassword | sudo tee -a /etc/nginx/.credentials.list
rm /etc/nginx/sites-enabled/default
```
Copy `webdav.conf` to `/etc/nginx/webdav.conf`


## WEBDAV client
* VM size = A1
* Ubuntu 18.04 server

### Configuration script
```
cat <<EOF | sudo debconf-set-selections
davfs2 davfs2/suid_file boolean false
EOF
sudo apt install -y davfs2

mkdir -p /mnt/webdav
WEBDAV_SERVER_URL=http://10.0.0.5/
sudo mount -t davfs ${WEBDAV_SERVER_URL} /mnt/webdav
```
