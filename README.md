# swarm01 apache-php

### Ref awaresome-compose
- https://github.com/docker/awesome-compose/tree/master/apache-php

### Wakatime project
- https://wakatime.com/@spcn19/projects/jseztswsjz?start=2023-02-18&end=2023-03-12

### Url apache-php
- https://spcn19apache.xops.ipv9.xyz/

### ขั้นตอนการติดตั้ง และใช้งาน ใน VM
 1. Set Template 

    - set time
      ```
      timedatectl set-timezone Asia/Bangkok
      ```

    - install Docker
      ```
      apt update; apt upgrade -y #อัปเดตแพ็คเกจภายในเครื่อง

      apt-get install ca-certificates curl wget gnupg lsb-release -y #ติดตั้งแพ็คเกจ

      mkdir -m 0755 -p /etv/apt/keyrings

      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg #ดาวโหลดไฟล์แพ็คเกจ Docker

      echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" |  tee /etc/apt/sources.list.d/docker.list > /dev/null

      apt-get update #อัปเดทไฟล์แพ็คเกจเพื่อไว้สำหรับให้ติดตั้ง
      apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y #ติดตั้ง Docker

      reboot
      ```

 2. Clone Templete ออกมา 3 Node คือ
    - manage
    - work1
    - work2

 3. Set Hostname
    ```
    hostnamectl set-hostname "ชื่อ Hostname โดยต้องห้ามซ้ำ" #spcn19-swarm01
    ```

 4. Reset Machine ID เพื่อขอ Public IP จาก DHCP 
    ```
    cp /dev/null /etc/machine-id
    rm /var/lib/dbus/machine-id
    ln -s /etc/machine-id /var/lib/dbus/machine-id
    init 0
    ```

 5. [ทำการเตรียม stack swarm and Portainer CE](#stack-swarm)
 6. [ทำการเตรียม Revert Proxy](#revert-proxy)
 7. [สร้าง Image จาก Dockerfile](#create-image-on-dockerfile)
 8. ทำการเตรียมไฟล์ docker-compose.yml สำหรับ Cluster Swarm #APPNAME => `spcn19apache`
    - version => เวอร์ชั่นของไฟล์ compose ต้อง 3 ขึ้นไป
    - services :
      - server : => ชื่อของ application
        - image => image ที่ต้อง build `thanawat1303/apache2-php-index`
        - networks => เน็ตเวิร์คของ Traefik
        - logging => ประวัติการทำงานของ container
          - driver => json-file คือ เลือกประเภทการ log เป็น json
        - volumes => ส่วนของการเก็บข้องมูลของ containner
          - path ที่เก็บข้อมูลภายใน host : path ที่เก็บข้อมูลภายใน container
        - container_name => ชื่อ container
        - deploy => เซ็ตการ deploy สำหรับ swarm
          - replicas => กำหนดเครื่อง worker ที่ต้องการให้ deploy containner ลงไป
          - labels => กำหนด labels ให้ application โดยจะเป็นการตั้งค่าเชื่อมต่อกับ Traefik
            - traefik.docker.network => ชื่อ network ของ Traefik
            - traefik.enable => กำหนดสถานะการใช้งาน
            - traefik.constraint-label => เลือก traefik ที่ต้องการให้ container ไปทำงาน
            - traefik.http.routers.${APPNAME}-https.entrypoints => กำหนด port ในการเชื่อมต่อเมื่อมีคำขอเข้าไปที่ traefik
            - traefik.http.routers.${APPNAME}-https.rule=Host(`${APPNAME}.xops.ipv9.xyz`) => กำหนด Domain ในการเข้าถึง application
            - traefik.http.routers.${APPNAME}-https.tls.certresolver => กำหนดการสร้างใบรับรอง url
            - traefik.http.services.${APPNAME}.loadbalancer.server.port => กำหนดให้มีการ balance ในการร้องขอ port ที่ container ทำงาน
            - traefik.http.routers.${APPNAME}-https.tls => เปิดใช้งาน Protocal TLS
          - resources => กำหนดสเปคที่ต้องการของ Containner
            - reservations => กำหนดค่าขั้นต่ำของสเปค
            - limits => กำหนดค่าสูงสุดของสเปค
    - networks => กำหนด networks ที่อยู่ภายในระบบ
      - webproxy => บริการ network revert proxy ที่อยู่ภายในระบบ
        - external => กำหนดสถานะของ network ที่อยู่ภายใน host
    - volumes => พื้นที่เก็บข้อมูลที่จะสร้างไว้ให้อยู่บน Host
      - app => ชื่อพื้นที่เก็บข้อมูล ภายใน host ต้องตรงตามที่กำหนดที่ volumes ที่ mount กับ contianner
 9. ทดลอง Deploy docker-compose ด้วย image ที่สร้าง บน Cluster ของตนเอง
 10. ทำการ Remote และ upload ไฟล์งานเข้าสู่ Repo swarm01 บน github
 11. ทำการนำข้อมูลในไฟล์ docker-compose หรือ LINK repo github เข้ากับ portainer ของระบบ
 12. Deploy

### Create Image on Dockerfile
 1. จัดการไฟล์ index.php ใน path app/index.php สำหรับ UI ใน application
 2. แก้ไขไฟล์ Dockerfile จากตัวอย่างให้เป็นแบบ Dockerfile ใน path app
 3. สร้าง Image จาก Dockerfile ใน path app ด้วยคำสั่ง
 
    ```
    docker build . -t <usernameDockerHub>/<repo>:<tag> #หากไม่ใส่ tag จะเป็น latest thanawat1303/apache2-php-index:v1
    ```
 4. push Image to DockerHub

     ```
     docker push <image ID> <usernameDockerHub>/<repo>:<tag> #หากไม่ใส่ tag จะเป็น latest thanawat1303/apache2-php-index:v1
     ```

### Stack Swarm
<a name="stack-swarm"></a>

 - Manager Swarm

   - Swarm init
     ```
     docker swarm init #รันในเครื่อง Manage
     ```

   - นำ Token Url ไป run บน worker ทุก Node ที่ต้องการให้เชื่อมต่อ

   - Check Node Stack swarm
     ```
     docker node ls
     ```

   - install portainer CE
     ```
     curl -L https://downloads.portainer.io/ce2-17/portainer-agent-stack.yml -o portainer-agent-stack.yml
     docker stack deploy -c portainer-agent-stack.yml portainer
     ```

   ### Ref
   - https://github.com/pitimon/dockerswarm-inhoure#swarm-init

### Revert Proxy
<a name="revert-proxy"></a>

 - Manager Traefik

   - Set IP สำหรับเครื่อง Client
     - แก้ไขไฟล์ hosts
       - windows C:\Windows\System32\drivers\etc\hosts
       - Linux /etc/hosts
     - เพิ่ม Domain ให้แต่ละโปรแกรมโดยเชื่อมเข้าสู่ IP ของ manager เช้น "ip manage" traefik.demo.local

   - สร้าง Network ใหม่
     ```
     docker network create --driver=overlay traefik-public
     ```

   - Get ID Node 
     ```
     export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}') 
     echo $NODE_ID
     ```

   - สร้าง Label ของ Node Manage
     ```
     docker node update --label-add traefik-public.traefik-public-certificates=true $NODE_ID
     ```

   - set Treafik
     ```
     export EMAIL=user@smtp.com
     export DOMAIN=<ชื่อ traefik domain ที่ต้องการให้เข้าถึง traefik>
     export USERNAME=admin
     export PASSWORD=<รหัสผ่าน traefik>
     export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
     echo $HASHED_PASSWORD
     ```

   - deploy traefik stack
     ```
     docker stack deploy -c traefik-host.yml traefik
     ```
     
   - ทดลองเปิดหน้า Dashboard Traefik

   ### Ref

   - https://github.com/pitimon/dockerswarm-inhoure/tree/main/ep03-traefik

### Remote Repo on LINUX
 1. ทำการสร้างไฟล์ README.md ใน Repo 
 2. git clone "URL GIT Repo"