Implementing-Wordpress-Web-Solution

 ## STEP1 Preparing Web Server

 - Create a EC2 instance server on AWS 

 - On the EBS console, create 3 storage volumes for the instance. This serves as additional external storage to our EC2 machine

 ![created_volumes](./images/created%20volumes.png)

 - Attach the created volumes to the EC2 instance 

 ![attach](./images/attached%20volumes.png)

 - SSH into the instance and on the EC2 terminal, view the disks attached to the instance. This is achieved using the `lsblk` command.

 ![show_attached_disks](./images/lsblk.png)


 - To see all mounts and free spaces on our server

 ![displaying_mountpoints](./images/mount.png)

 - Create single partitions on each volume on the server using `gdisk `

 ![creating_partitions](./images/psrtition.png)
 ![partitioned](./img/7.partitioned.jpg)


 - Installing LVM2 package for creating logical volumes on a linux server.

 ![lvm2_installation](./images/lvm%20yum%20install.png)

 - Creating Physical Volumes on the partitioned disk volumes <br/>
 `sudo pvcreate <partition_path>`

 ![marking_physical_volumes](./images/pvcreate.png)


 - Next we add up each physical volumes into a volume group <br/>
 `sudo vgcreate <grp_name> <pv_path1> ... <pv_path1000> `

 ![creating_volume_groups](./images/vg%20create.png)

 - Creating Logical volumes for the volume group <br/>
 `sudo lvcreate -n <lv_name> -L <lv_size> <vg_name>`

 ![creating_logical_volumes](./images/lv%20create.png)


 - Our logical volumes are ready to be used as filesystems for storing application and log data.
 - Creating filesystems on the both logical volumes

 ![file_systems](./images/mount.png)


 - The apache webserver uses the html folder in the var directory to store web content. We create this directory and also a directory for collecting log data of our application

 ![required_directory_creation](./images/recovery.png)

 - For our filesystem to be used by the server we mount it on the apache directory . Also we mount the logs filesystem to the log directory

 ![mounting_syncing](./images/mount1.png)

 - Mount logs logical volume to var logs



 - Restoring back var logs data into var logs

 ![syncing_back to varlogs](./images/sync.png)

 ## Persisting Mount Points
 - To ensure that all our mounts are not erased on restarting the server, we persist the mount points by configuring the `/etc/fstab` directory

 - `sudo blkid` to get UUID of each mount points

 ![uuid_update](./images/blkld.png)

 - `sudo vi /etc/fstab` to edit the file

 ![persisting_mount_config](./images/mount%20persist.png)

  - testing mount point persistence

 ![testing config](./images/test%20config.png)

 ## STEP2 Preparing DataBase Server
  - Repeated all the steps taken to configure the web server on the db server. Changed the `apps-lv` logical volume to `db-lv`

 ![configuration on db server](./images/test%20config.png)

 ## STEP3 Configuring Web Server
 - Run updates and install httpd on web server
 ```
 yum install -y update
 sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
 ```

 - Start web server

 ![starting_web_server](./images/apache%20status.png)

 - Installing php and its dependencies
 ```
 sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
 sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
 sudo yum module list php
 sudo yum module reset php
 sudo yum module enable php:remi-7.4
 sudo yum install php php-opcache php-gd php-curl php-mysqlnd
 sudo systemctl start php-fpm
 sudo systemctl enable php-fpm
 setsebool -P httpd_execmem 1
 ```

 - Restarting Apache: 
 `sudo systemctl restart httpd`

 - Downloading wordpress and moving it into the web content directory
 ```
 mkdir wordpress
 cd   wordpress
 sudo wget http://wordpress.org/latest.tar.gz
 sudo tar xzvf latest.tar.gz
 sudo rm -rf latest.tar.gz
 cp wordpress/wp-config-sample.php wordpress/wp-config.php
 cp -R wordpress /var/www/html/
 ```

 - Configure SELinux Policies
 ```
 sudo chown -R apache:apache /var/www/html/wordpress
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 sudo setsebool -P httpd_can_network_connect=1
 ```
 - Starting database server

 ![starting_db_server](./images/db%20server.png)

 ## STEP4 Installing MySQL on DB Server
 ```
 sudo yum update
 sudo yum install mysql-server
 ```

 To ensure that database server starts automatically on reboot or system startup
 ```
 sudo systemctl restart mysqld
 sudo systemctl enable mysqld
 ```

 ## STEP5 Setting Up DB Server
 ![setting_up_db](./images/db%20server.png)

  - Ensure that we add port `3306` on our db server to allow our web server to access the database server.

 ![security_grp_db](./images/secure%203306.png)

 ## Connecting Web Server to DB Server

 Installing mySQl client on the web server so we can connect to the db server

 ```
 udo yum install mysql
 sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
 ```
 ![connecting_to_db_from_web](./images/connecting%20.png)

 - On the web browser, access web server using the public ip address of the server 

 ![Successful_connection](./images/wordpress%20page.png)