# Own Your Stream --- Toolkit For Online Freedom

Follow the steps below to set up your own streaming server. 

![alt text](https://github.com/Better-Call-Jason/nginx-streaming-server-app/blob/main/episode1-thumbnail2.png?raw=true)

## Table of Contents

1. [Getting Started](#Prerequisites)
2. [Set Up](#Installation)
3. [Build Your Receiver](#receiver)
4. [Build Your Broadcaster](#broadcaster)
5. [Build Your Viewer](#viewer)
6. [Donate](#Donate)

## <a name='Prerequisites'></a>Getting Started

- download obs : https://obsproject.com
- purchase domain : https://domains.squarespace.com
- create cloud account  
    - create linode account : https://www.linode.com  
    - create aws account : https://aws.amazon.com
- access your cloud account <br />
    - linode account : https://cloud.linode.com/linodes<br />
    - aws/lightsail : https://lightsail.aws.amazon.com/ls/webapp/home/instances<br />
    - create your instance with  `ubuntu 20.04`<br />
- point your domain to your created instance's ip
    - navigate to your domain dns
    - create an A record
    - enter a subdomain or leave blank
    - address - enter instance ip address
    - TTL - 60 seconds or minimum
  
## <a name='Installation'></a>Set Up

SSH into your instance and let's begin setting up your streaming server, we'll first update the instance and then install our server application called NGINX

- update instance  
    ```sudo apt update```  <br />
    ```sudo apt upgrade``` <br /><br />
- install nginx - server application  
    ```sudo apt install nginx```<br /><br />
- check status  
    ```systemctl status nginx```  
    `ctl` + `c` to exit the server
    
If Nginx is running correctly, you will see a green Active status.

## <a name='receiver'></a>Build Your Receiver

Now, let's set up your receiver so your server can receive your live stream signal. 

- install nginx rmtp module, our receiver module  
    ```sudo apt install libnginx-mod-rtmp```<br /><br />
- restart nginx   
    ```sudo systemctl restart nginx```<br />
    ```systemctl status nginx```  <br />
    `ctl + c` to exit

- configure nginx to receive stream  
    ```sudo nano /etc/nginx/nginx.conf```  
- copy and insert this code below at the bottom of the page<br />

```
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;
            # allow publish YOUR_LOCAL_IP_ADDRESS;
            # deny publish all;

            # Setup HLS
            hls on;
            hls_path /mnt/hls/;
            hls_fragment 3;
            hls_playlist_length 60;
            #deny play all;
            allow play all;
        }
    }
}
```
- after pasting click `ctl` + `s` and then `ctl` + `x`<br />

```
git clone https://github.com/username/project.git
cd project
npm install
```

## <a name='Code'></a> Code Examples

Here's an example of some code used in this project:

```javascript
function() {
  console.log("Hello, World!");
}
```

## <a name='Donate'></a>Donate

If you found this project helpful and would like to contribute to its ongoing development, consider making a donation.

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](YOUR_PAYPAL_LINK)

Thanks for your support!
