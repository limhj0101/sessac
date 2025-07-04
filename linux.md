vagrant, github, vscode

git연습

https://learngitbranching.js.org/?locale=ko

수업 강사님 드라이브

https://drive.google.com/drive/folders/1ypIAcyVIDbFt-tgSKGHhIDFyrNw4nqyz



`print`

```bash
#!/bin/bash
echo "hello"
```

vagrantfile (1)
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168.56."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "sessac1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "sessac1"
      vb.cpus = 2
      vb.memory = 4096
    end

    node.vm.network "private_network", ip: vm_subnet + "11", nic_type: "virtio"
    node.vm.hostname = "sessac1"
  end
end
```

vagrantfile(2)
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
#vm_image = "nobreak-labs/rocky-9"
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168.56."

install_tools_script = <<-SCRIPT
# lsd
LSD_VERSION="1.1.5"
curl -LO https://github.com/lsd-rs/lsd/releases/download/v${LSD_VERSION}/lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu.tar.gz
tar -xzf lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu.tar.gz
mv lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu/lsd /usr/local/bin/
rm -rf lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu.tar.gz lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu

# bat
BAT_VERSION="0.24.0"
curl -LO https://github.com/sharkdp/bat/releases/download/v${BAT_VERSION}/bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu.tar.gz
tar -xzf bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu.tar.gz
mv bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu/bat /usr/local/bin/
rm -rf bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu.tar.gz bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu

# Set aliases
ALIASES="
alias ls='lsd'
alias ll='lsd -l'
alias la='lsd -a'
alias lla='lsd -la'
alias cat='bat -pp'
"

# Add aliases to vagrant
echo "$ALIASES" >> /home/vagrant/.bashrc
chown vagrant:vagrant /home/vagrant/.bashrc

# Add aliases to root
echo "$ALIASES" >> /root/.bashrc

# Add aliases to skel for new users
echo "$ALIASES" >> /etc/skel/.bashrc

# Apply bashrc
source /root/.bashrc
sudo -u vagrant bash -c 'source /home/vagrant/.bashrc'
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "sessac1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "sessac1"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "11", nic_type: "virtio"
    node.vm.hostname = "sessac1"
    node.vm.provision "shell", inline: install_tools_script, name: "install_tools"
  end
end
```

# 1장
## 사용자 생성
```
    1  cat /etc/passwd
    2  cat /etc/shadow
    3  sudo cat /etc/shadow
    4  cat /etc/group
    5  cat /etc/gshadow
    6  sudo cat /etc/gshadow
    7  ls -l vagrant
    8  ls -l
    9  ls -l /etc
   10  ls man
   11  man ls
   12  alias ll
   13  useradd user01
   14  su - useradd user01
   15  su -
   16  sudo useradd user01
   17  tail -1 /etc/passwd
   18  tail -1 /etc/shadow
   19  sudo tail -1 /etc/shadow
   20  passwd user01
   21  sudo passwd user01
   22  sudo tail -1 /etc/shadow
   23  mkdir /home/guest
   24  sudo mkdir /home/guest
   25  sudo useradd -D -b /home/guest
   26  sudo useradd -D
   27  sudo useradd user02
   28  sudo tail -1 /etc/passwd
   29  sudo useradd -u 2000 -g 10 -m -d /home/guest/user03 -s /bin/sh user03
   30  sudo tail -1 /etc/passwd
   31  history
   ```

# 5장
## 파일시스템

```
    3  sudo fdisk /dev/sd
    4  sudo fdisk /dev/sdb
    5  lsblk
    6  sudo partprobe
    7  sudo partprobe /dev/sdb
    8  ls -li /etc/passwd
    9  lsblk
   10  sudo fdisk /dev/sdb
   11  sudo partprobe /dev/sdb
   12  lsblk
   13  sudo mkfs.xfs /dev/sdb1
   14  sudo file -s /dev/sdb1
   15  lsblk
   16  sudo mkswap /dev/sdb2
   17  sudo file -s /dev/sdb2
   18  sudo file -s /dev/sdb1
   19  sudo blkid
   20  ls -l /dev/sdb2
   21  ls -l /dev
   22  sudo mkdir /mnt/xfsdata
   23  sudo mount /dve/sdb1 /mnt/xfsdata
   24  sudo systemctl daemon-reload
   25  df -h
   26  mount
   27  mount | grep sdb1
   28  sudo mount /dev/sdb1 /mnt/xfsdata
   29  df -h
   30  mount | grep sdb1
   31  sudo touch a /mnt/xfsdata
   32  ls -l /dev/sdv1
   33  ls -l /dev/sdb1
   34  ll -f /dev/sdb1
   35  ll /dev/sdb1
   36  sudo touch /mnt/xfsdata/a
   37  ls -l /mnt/xfsdata
   38  sudo vi /etc/fstab
   39  sudo umount /mnt/xfsdata
   40  mount | grep sdb
   41  sudo mount -a
   42  df -h | grep sdb1
   43  sudo mount | grep sd
   44  reboot
   45  sudo reboot
   46  ls -l /mnt/xfsdata
   47  mount | grep sdb
   ```

# SQL
```
USE sample_db;

SELECT * FROM stay;

-- 사용자 ID가 'CHO04'인 고객의 이름과 전화번호
SELECT user_nm, phone
FROM user
WHERE user_id = 'CHO04';

SELECT *
FROM user
WHERE addr = '서울';

SELECT * 
FROM user
WHERE age = '31';

SELECT *
FROM user
WHERE age > '31';

SELECT 'A' = 'a';

SELECT user_nm, phone
FROM user
WHERE addr = '부산';


SELECT stay_title, price
FROM stay;

SELECT price, stay_title 
FROM stay;

SELECT price, price, price
FROM stay;

SELECT user_nm AS 고객명, phone AS 연락처
FROM user
WHERE addr = '서울';

SELECT user_nm 고객명, phone 연락처
FROM user
WHERE addr = '서울';


SELECT addr FROM user;
SELECT DISTINCT addr FROM user;
SELECT type FROM stay;
SELECT DISTINCT type FROM stay;


SELECT DISTINCT addr, age FROM user;

SELECT COUNT(DISTINCT addr) FROM user;

SELECT  10 + 20;
SELECT 100*1.1 AS '부가세포함';
SELECT CONCAT('hello', ' world')


SELECT NOW();
SELECT CURDATE();
SELECT CURTIME();



SELECT * FROM user ORDER BY age;
SELECT * FROM user ORDER BY age DESC;
SELECT stay_title, price FROM stay ORDER BY price DESC;

-- 지역별로 정렬 후, 같은 지역 내에서는 나이 순
SELECT user_nm, addr, age
FROM user
ORDER BY addr, age;

SELECT user_nm, age
FROM user
WHERE addr = '서울'
ORDER BY age;


-- .
SELECT * FROM user LIMIT 5;

-- 가장 저렴한 숙소
SELECT stay_title, price
FROM stay
ORDER BY price
LIMIT 5;

SELECT stay_title, price
FROM stay
ORDER BY price
LIMIT 5 OFFSET 2;


SELECT user_nm FROM user WHERE addr !='부산';
SELECT user_nm FROM user WHERE addr <>'부산';

SELECT user_nm, age FROM user WHERE age >= 20;



-- 28세 이상이면서 부산에 거주하는 고객(AND)
SELECT user_nm, addr, age
FROM user
WHERE age >= 28 AND addr = '부산';

-- 부산 또는 여수에 거주하는 고객(OR)
SELECT user_nm, addr
FROM user
WHERE addr = '부산' OR addr = '여수';


SELECT * FROM user WHERE phone = NULL;
SELECT * FROM user WHERE phone != NULL;
SELECT NULL = NULL;
SELECT NULL != NULL;
SELECT NULL > NULL;
SELECT 5 * NULL;

SELECT user_nm, phone FROM user WHERE phone IS NULL;
SELECT user_nm, phone FROM user WHERE phone IS NOT NULL;

-- .
SELECT stay_title, price
FROM stay
WHERE price >= 150000 AND price <= 250000;

SELECT stay_title, price
FROM stay
WHERE price BETWEEN 150000 AND 250000;

SELECT user_nm FROM user
WHERE user_nm BETWEEN '김' AND '최';

SELECT stay_title, price
FROM stay
WHERE price NOT BETWEEN 150000 AND 250000;


SELECT user_nm FROM user
WHERE user_nm LIKE '김%';

SELECT user_nm FROM user
WHERE user_nm LIKE '%영%';

SELECT user_nm FROM user
WHERE user_nm LIKE '%수';

SELECT user_nm FROM user
WHERE user_nm LIKE '김__';

SELECT user_nm FROM user
WHERE user_nm NOT LIKE '김%';


-- .

SELECT COUNT(*) FROM user;

SELECT COUNT(*) AS 전체고객수,
COUNT(phone) AS 전화번호고객수,
COUNT(email) AS 이메일고객수
FROM user;

SELECT AVG(age) AS 평균나이
FROM user;

SELECT AVG(price) AS 평균가격
FROM stay;

-- . GROUP BY 기본 문법
SELECT 그룹컬럼, 집계함수(컬럼)
FROM 테이블명
GROUP BY 그룹컬럼;

-- 지역별로 고객이 몇명 씩 있는지 조회
SELECT 
	addr AS 지역,
	COUNT(*) AS 고객수
FROM user
GROUP BY addr;

-- 지역별로 고객이 몇명 씩 있는지 조회
SELECT 
	type AS 숙소타입,
	COUNT(*) AS 숙소갯수
FROM stay
GROUP BY type;

-- 30세 이상 고객들의 지역별 분석 현황
SELECT 
	addr AS 자역,
	COUNT(*) AS '30세 이상 고객수'
FROM user
WHERE age >= 30
GROUP BY addr;

-- 고객 수가 많은 지역 순으로 정렬
SELECT 
	addr AS 지역,
	COUNT(*) 고객수
FROM user
GROUP BY addr 
ORDER BY COUNT(*) DESC;


-- WHERE : 지역별 나이가 30세 이상인 고객수
SELECT addr, COUNT(*)
FROM user
WHERE age >= 30
GROUP BY addr;

-- 고객수가 2명 이상인 지역의 고객수
SELECT addr, COUNT(*)
FROM user
GROUP BY addr
HAVING COUNT(*) >= 2;

-- 고객이 2명 이상인 지역을 고객수 순으로 정렬
SELECT addr AS 지역, COUNT(*) AS 고객수
FROM user
GROUP BY addr
HAVING COUNT(*) >= 2
ORDER BY COUNT(*) DESC;










-- 연습문제

부산에 거주하는 고객
나이가 25세 이상인 고객
강릉에 위치한 숙소들
가격이 150000원 이하인 숙소

—

SELECT user_nm, addr
FROM user
WHERE addr = '부산';

SELECT user_nm, age 
FROM user
WHERE age >= 25;

SELECT stay_title, location
FROM stay
WHERE location = '강릉';

SELECT stay_title, price
FROM stay
WHERE price <= 150000;



모든 고객의 이름과 이메일 조회
서울 거주 고객의 이름과 전화번호만 조회
모든 숙소의 이름과 위치만 조회
제주에 위치한 숙소의 이름, 가격, 수용인원만  조회

—

SELECT user_nm, email
FROM user;

SELECT user_nm, phone
FROM user;

SELECT stay_title, location
FROM stay;

SELECT stay_title, price, capacity
FROM stay
WHERE location = '제주';



모든 고객의 이름을 ‘고객명’, 나이를 ‘연령’ 으로 조회
경주에 위치한 숙소의 제목을 ‘숙소명’, 가격을 ‘1박 요금’으로 조회
부산 거주 고객의 이름을 '이름', 전화번호를 '연락처', 이메일을 '이메일 주소’ '로 조회

—

SELECT user_nm AS 고객명, age AS 연령
FROM user;

SELECT stay_title AS 숙소명, price AS '1박 요금'
FROM stay;

SELECT user_nm AS 이름, phone AS 연락처, email AS '이메일 주소'
FROM user
WHERE addr ='부산';



고객들의 나이대를 중복 없이 조회
숙소들의 수용인원을 중복 없이 조회
예약 상태 종류를 중복 없이 조회
고객 거주 지역을 중복 없이 조회하되, 가나다 순으로 정렬

—

SELECT DISTINCT age FROM user;
-- 오름차순으로 정렬
SELECT DISTINCT age FROM user ORDER BY age;

SELECT DISTINCT capacity FROM stay;
-- 오름차순으로 정렬
SELECT DISTINCT capacity FROM stay ORDER BY capacity;

SELECT DISTINCT resv_status FROM reservations;

SELECT DISTINCT addr FROM user ORDER BY addr ASC;



250000원에서 10% 할인한 금액 계산
현재 날짜와 시간을 '지금'이라는 별칭으로 조회
'여행을', ' ', '떠나요'를 연결해서 '메시지'라는 별칭으로 조회
오늘부터 크리스마스(12월 25일)까지 며칠 남았는지 계산[도전문제]

—

SELECT 250000 * 0.9;

SELECT NOW() AS '지금';

SELECT CONCAT('여행을',' ','떠나요') AS 메세지;

SELECT DATEDIFF('2025-12-25', '2025-07-03');



숙소를 수용인원이 많은 순서대로 정렬해서 조회
고객을 가입일이 최근 순서대로 정렬해서 이름과 가입일만 조회
제주에 위치한 숙소를 가격이 저렴한 순서대로 정렬해서 조회
고객을 지역별로 정렬하고, 같은 지역에서는 나이가 많은 순으로 정렬

—

SELECT stay_title, capacity
FROM stay
ORDER BY capacity DESC;

SELECT user_nm, join_date
FROM user
ORDER BY join_date DESC;

SELECT stay_title, price
FROM stay
WHERE location = '제주'
ORDER BY price ASC;

SELECT user_nm, addr, age
FROM user
ORDER BY addr ASC, age ASC;



나이가 32세 이상인 고객의 이름과 나이 조회
가격이 160000원 미만인 숙소의 제목과 가격
2022년도에 가입한 고객을 조회[도전문제](힌트: AND)

—

SELECT user_nm, age
FROM user
WHERE age >= 32;

SELECT stay_title, price
FROM stay
WHERE price < 160000;

SELECT *
FROM user
WHERE YEAR(join_date) = 2022;
-- LIKE 활용
SELECT *
FROM user
WHERE join_date LIKE '2022%';
-- 범위로 조회
SELECT *
FROM user
WHERE join_date >= '2022-01-01' AND join_date < '2023-01-01';


이메일이 등록되지 않은 고객의 이름과 이메일을 조회
전화번호가 등록된 30세 이상 고객의 이름, 나이, 전화번호를 조회

SELECT user_nm, email
FROM user
WHERE email IS NULL;

SELECT user_nm, age, phone
FROM user
WHERE age >= 30 AND phone IS NOT NULL;

```

## 7월 4일
```
-- 서브쿼리
SELECT stay_title, price
FROM stay
WHERE price > (SELECT AVG(price) FROM stay);

-- 서브쿼리 없이 구하기
SELECT AVG(price) FROM stay;
SELECT stay_title, price FROM stay
WHERE price > 179000;

SELECT user_nm, age
FROM user
WHERE age > (SELECT MIN(age) FROM user);

SELECT stay_title, price
FROM stay
WHERE stay_id IN (SELECT DISTINCT stay_id FROM reservations);

-- .
SELECT stay_title, price, location
FROM (SELECT * FROM stay WHERE price >= 200000) AS 프리미엄
WHERE location = '부산' OR location = '제주';

SELECT stay_title, price, location
FROM (SELECT * FROM stay WHERE price >= 200000) AS 프리미엄
WHERE location IN('부산','제주');

-- 성인(20살) 고객 중에서 특정 지역(서울) 거주자만 조회
SELECT user_nm, age, addr
FROM (SELECT * FROM user WHERE age >= 20) AS adult_users
WHERE addr = '서울';

-- 20만원 이상인 숙소들 중에서 부산이나 제주에 위치한 곳 중에 수용인원이 3명 이상인 숙소의 평균 가격을 조회

SELECT AVG(price)
FROM stay
WHERE price >= 200000 AND location IN('부산','제주') AND capacity >= 3;

SELECT AVG(price) AS 평균가격
FROM (SELECT * FROM stay WHERE price >= 200000) AS 프리미엄
WHERE location IN('부산','제주') AND capacity >= 3;

-- .
-- 지역별 평균 가격이 20만원 이상인 지역들 조회
SELECT location, 평균가격
FROM (
SELECT location, AVG(price) AS 평균가격 
FROM stay
GROUP BY location
)
AS location_stats WHERE 평균가격 >= 200000;

SELECT AVG(price), location AS 평균가격 
FROM stay
GROUP BY location;


SELECT stay_title, price
FROM stay
WHERE EXISTS(
	SELECT 1
	FROM reservations
	WHERE  reservations.stay_id = stay_id
);

SELECT *
FROM reservations
WHERE  reservations.stay_id = stay_id;



-- join

SHOW TABLES;
DESC customer;
SELECT * FROM customer LIMIT 5;

DESC orderitem;
SELECT * FROM orderitem LIMIT 5;

-- customer.CUST_NO = orderitem.CUST_NO

-- 고객 이름과 주문 정보 함께 보기
SELECT 
	c.CUST_name AS 고객명,
	c.area AS 고객지역,
	o.order_date AS 주문일자,
	o.total_amount AS 주문금액
FROM customer c 
JOIN orderitem o ON c.CUST_NO = o.CUST_NO;

-- 서울 지역 고객이 주문한 전자제품(category_id 'C001')의 내역을 조회하세요
-- 고객명, 상품명, 가격, 주문수량을 출력

SELECT 
	c.CUST_name AS 고객명,
	p.product_name AS 상품명,
	p.price AS 가격,
	o.order_qty AS 주문수량
FROM customer c JOIN orderitem o ON c.CUST_NO = o.CUST_NO
JOIN product p ON o.product_id = p.product_id;


-- 모든고객과 고객들의 주문 정보를 함께보기(주문하지 않은 고객도 포함)
SELECT 
	c.CUST_name AS 고객명,
	c.area AS 지역,
	o.order_date AS 주문일자,
	o.total_amount AS 주문금액
FROM customer c
LEFT JOIN orderitem o ON c.CUST_NO = o.CUST_NO;
```
