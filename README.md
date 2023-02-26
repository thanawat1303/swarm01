# swarm01 Apache2:PHP
### ขั้นตอนการติดตั้ง และใช้งาน ใน VM
 1. Set Template 

   set time

    timedatectl set-timezone Asia/Bangkok

   install Docker

    apt update; apt upgrade -y #อัปเดตแพ็คเกจภายในเครื่อง

    apt-get install ca-certificates curl wget gnupg lsb-release -y #ติดตั้งแพ็คเกจ

    mkdir -m 0755 -p /etv/apt/keyrings

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg #ดาวโหลดไฟล์แพ็คเกจ Docker

    echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" |  tee /etc/apt/sources.list.d/docker.list > /dev/null

    apt-get update #อัปเดทไฟล์แพ็คเกจเพื่อไว้สำหรับให้ติดตั้ง
    apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y #ติดตั้ง Docker

    reboot

 2. Set Hostname 

    ``` hostnamectl set-hostname "ชื่อ Hostname โดยต้องห้ามซ้ำ" => spcn19-swarm01 ```

 3. Reset Machine ID เพื่อขอ Public IP จาก DHCP
 4. ทำการนำ Url Token จากคำสั่ง 
 
    docker swarm init 
        
    ในเครื่อง Manage มาทำการรันเพื่อเชื่อมต่อ swarm

 5. ทำการเตรียมไฟล์ docker-compose.yml
    - image => ใช้ image จาก DockerFile
 6. ทำการ Remote ไฟล์งานเข้าสู่ Repo swarm01 บน github
 7. ทำการนำข้อมูลในไฟล์ docker-compose หรือ LINK repo github

### Remote Repo on LINUX
 1. ทำการสร้างไฟล์ README.md ใน Repo 
 2. git clone <URL GIT Repo>

### Ref

- https://github.com/docker/awesome-compose/tree/master/apache-php
