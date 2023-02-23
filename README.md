# swarm01
## ขั้นตอนการติดตั้ง และใช้งาน ใน VM
 1. Set Template 

   set Hostname
    
    hostnamectl set-hostname "ชื่อ Hostname โดยต้องห้ามซ้ำ"

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
      
 2. Reset Machine ID เพื่อขอ Public IP จาก DHCP
 3. ทำการนำ Url Token จากคำสั่ง 
 
        docker swarm init 
        
    ในเครื่อง Manage มาทำการรันเพื่อเชื่อมต่อ swarm

 4. ทำการเตรียมไฟล์ docker-compose.yml

### Remote Repo on LINUX
 1. ทำการสร้างไฟล์ README.md ใน Repo 
 2. git clone <URL GIT Repo>