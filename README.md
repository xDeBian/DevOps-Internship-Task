**DevOps Internship Task**

###### 1. Setup NFS client
    sudo apt update
    sudo apt install nfs-common
    sudo mkdir -p /opt/share

> - Create a Mount Point

    sudo mkdir -p /opt/share

> - Mount the NFS Share

    sudo mount -t nfs nfs.devops.tbc:/opt/share /opt/share
![1](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/d9704e12-2f33-4f66-a6dc-a471a4c06465)

> Extract **target.iso** file (message: you did it) :D

![image003](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/aa84737b-2896-4819-88fc-578d377aa437)

------------


###### 2. FInd Random files

> I Used Biglogfile.Log To See The Current Process And What Was Created Automatically After Deletion: **Auditctl** And **Inotifywait**

> **/usr/share/javascript**
**/etc/apache2/conf-available**


> We delete: **deleteme, iamhere.iso, heretoo.zip, biglogfile.log, aws**
+5GB free space :D

![image007](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/5758fa6c-d620-457e-87cf-a08fc8dae212)

------------


![image009](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/a6438027-3485-4266-b550-554e64d4f450)

------------


![image011](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/3dd62463-15eb-4fb1-b037-e75edfb0f87a)

------------


###### 3. Resize *var* and *root* Volume

![image](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/988c0f74-c298-4ee4-a562-00182465ff6b)


```shell
sudo pvcreate /dev/sda4
sudo vgextend ubuntu /dev/sda4


```
 
> Let'S Increase **Var** By 6GB, Since Its Capacity Is 12GB. 


  ```shell
sudo lvextend -l +6G /dev/ubuntu/var
sudo resize2fs /dev/ubuntu/var

```
> Let's increase the **root** with the remaining space


    lvextend -l +100%FREE /dev/ubuntu/root
    resize2fs /dev/ubuntu/root
 
![image015](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/33be523b-36c5-4e40-8bc6-e8f8680e8f6d)


------------


**4. Block any TCP traffic towards 8000 port.**
> To test port 8000, first install Nginx and proxy port:80 to 8000.
 > Delete from **/etc/modprobe.d/iptables-blacklist.conf** file

- install ip_tables /bin/false
- install nf_tables /bin/false
- install x_tables /bin/false

```shell
sudo iptables -A INPUT -p tcp --dport 8000 -j DROP
```
![image021](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/8ce1eb0b-97e2-49bb-b0e2-99cf2e4a5a2c)



> After the restart, if port 8000 is blocked, let's make a service

`/etc/iptables-rules.sh`



    #!/bin/bash
    iptables -A INPUT -p tcp --dport 8000 -j DROP
    
    


`/etc/systemd/system/iptables-rules.service`

```shell
[Unit]
Description=Apply iptables rules

[Service]
Type=oneshot
ExecStart=/etc/iptables-rules.sh

[Install]
WantedBy=multi-user.target
```
> It was possible to use **UFW**



------------


**5.Host Flask Web App Using NGINX and Gunicorn**

>Let's make a clone  http://gitlab.devops.tbc/shvanadze/uptime-webapp

- Let's write our server data to an** .env** file
- Install **pip** after the **requirements** libraries
- Run the **app.py** file
- Let's check the application

> Create a **gunicorn_config.py** config in the /root/ folder and add it.



    bind = "0.0.0.0:8000"
    workers = 2

> Let's create **flaskapp.service**

```shell
[Unit]
Description=Gunicorn instance to serve uptime-webapp
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/uptime-webapp
ExecStart=/usr/local/bin/gunicorn --config /root/uptime-webapp/gunicorn_config.py app:app

[Install]
WantedBy=multi-user.target
```
> Let's do nginx proxy **80 to 8000**

```shell
server {
    listen 80 default_server;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;  # Forward requests to port 8000
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}





```
![image035](https://github.com/xDeBian/DevOps-Internship-Task/assets/20703661/061859cf-73ba-4816-a6ae-31e0af8c63ab)

> Create a **cron** task that will check for updates **every 1 day**
Let's create **:check_update.sh** in the root folder

```shell
#!/bin/bash

# Navigate to repository directory
cd /root/uptime-webapp

# Fetch the latest changes from the remote repository
git fetch origin

# Check if the main branch has been updated
if git diff --quiet origin/main; then
    echo "No updates found."
else
    echo "Updates found. Pulling changes..."
    git pull origin main
fi

```

------------
** 6. Create a Private repository on gitlab.devops.tbc **

[http://gitlab.devops.tbc/ddzneladze/ansible](http://gitlab.devops.tbc/ddzneladze/ansible "http://gitlab.devops.tbc/ddzneladze/ansible")

------------
