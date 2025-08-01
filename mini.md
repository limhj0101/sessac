# 미니프로젝트
## 1번
### vagrant 파일

```

# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "web1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "web1"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "56.44", nic_type: "virtio"
    node.vm.network "private_network", ip: vm_subnet + "57.44", nic_type: "virtio"
    node.vm.hostname = "web1"
  end

  config.vm.define "web2" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "web2"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "56.45", nic_type: "virtio"
    node.vm.network "private_network", ip: vm_subnet + "57.45", nic_type: "virtio"
    node.vm.hostname = "web2"
  end
end
```

### 도구 설치
웹 서버, mysql, php 설치
```
sudo yum install -y httpd mysql mysql-server php php-mysqlnd
```
wordpress 압축파일 다운로드
```
curl -o wordpress.tar.gz https://wordpress.org/latest.tar.gz
```
혹은 
```
wget https://wordpress.org/latest.tar.gz
```
아파치 db연결
```
sudo setsebool -P httpd_can_network_connect_db on
```

### 웹 서버 WP 설정
웹 서버
```
sudo systemctl start httpd
sudo systemctl enable httpd
sudo firewall-cmd --add-server=http --permanent
sudo firewall-cmd --reload
```
WP 압축해제
```
sudo tar -xvf wordpress.tar.gz -C /var/www/html
```
WP 설정파일에 db정보 입력

-압축 해제하면 있는 sample.php 파일을 sample 지우고 복사하여 설정
```
sudo cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php

sudo vi /var/www/html/wordpress/wp-config.php
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wp' );

/** Database username */
define( 'DB_USER', 'wp-user' );

/** Database password */
define( 'DB_PASSWORD', 'P@ssw0rd' );

/** Database hostname */
define( 'DB_HOST', '192.168.56.44' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```
가상 호스트 설정
```
sudo vi /etc/httpd/conf.d/wordpress.conf

<VirtualHost *:80>
        ServerName example.com
        DocumentRoot    /var/www/html/wordpress

        <Directory      "/var/www/html/wordpress">
                AllowOverride All
        </Directory>

</Virtualhost>
```
설정 적용
```
sudo systemctl restart httpd
```

### DB설정
```
sudo systemctl start mysqld
sudo mysql

mysql> create database wp;
mysql> create user 'wp-user'@'192.168.56.44' identified by 'P@sswOrd';
mysql> grant all privileges on wp.* to 'wp-user'@'192.168.56.44'

exit
```
### 검증
```
curl -I http://192.168.56.44
```
브라우저 접속


## 2번
