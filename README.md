# Own Your Stream --- Toolkit For Online Freedom Episode 1

Follow the steps below to set up your own streaming server. 

![alt text](https://github.com/Better-Call-Jason/nginx-streaming-server-app/blob/main/episode1-thumbnail2.png?raw=true)

## Table of Contents

1. [Getting Started](#Prerequisites)
2. [Set Up](#Installation)
3. [Build Your Receiver](#receiver)
4. [Build Your Broadcaster](#broadcaster)
5. [Add Security](#ssl)
6. [Configure OBS](#obs)
7. [Configure Analytics and Firewall](#tcpdump)
9. [Build Your Viewer](#viewer)
10. [Donate](#Donate)

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

## <a name='broadcaster'></a>Build Your Broadcaster

Now, let's set up your broadcaster so your server can beam your live stream signal to surfers on your website. 

- configure nginx to broadcast the stream  
    `sudo nano /etc/nginx/sites-available/default`

delete all existing code and paste the code below :

```bash
server {

      listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html;

    server_name your domain name;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ ^/live/?$ {
        rewrite ^/live/?$ /stream.html break;
    }

    location /hls {
        types {
          application/vnd.apple.mpegurl m3u8;
          video/mp2t ts;
     }
     alias /mnt/hls/;
     expires -1;
     add_header 'Cache-Control' 'no-cache';
     }
 }
```
- after pasting click `ctl` + `s` and then `ctl` + `x`<br />

## <a name='ssl'></a>Add Security

Now, let's set up the security of your server and application. These next steps require that you've purchased a domain and followed the steps to set up an A record. It can take a while for this to work. So if it fails, give it some time before trying again.

- install certbot and let's encrypt  
    `sudo apt-get install certbot python3-certbot-nginx`
    
- create the ssl for your domain  
    `sudo certbot --nginx -d your domain name`
    
- automate the renewal of your domain ssl  
    `sudo certbot renew --dry-run`
    
   - add a cron job to manage the renewal  
    `sudo crontab -e`
    
   - enter this line at the bottom and save and exit  
    `0 12 */10 * * /usr/bin/certbot renew --quiet`
    
   - see active scheduled cron jobs  
    `sudo crontab -l`
- recheck nginx after certbot completes  
`sudo nano /etc/nginx/sites-available/default`

should see changes and 443 code added

## <a name='obs'></a>Configure OBS

Now, let's set up your OBS software to broadcast to your receiver

- Open OBS Studio on your PC or Mac.
    
- Click on `Settings` in the bottom right of the OBS Studio window.
    
- In the settings panel, click on `Stream`.
    
- Here, under `Service`, select `Custom...`.
    
- In the `Server` field, enter the RTMP URL of your Nginx server. The URL would be in the format: `rtmp://<Your_Server_IP>/live`.
    
- set your stream key, that will be needed later.  
    `YOUR_STREAM_KEY`
    
- Click on `Apply` and then `OK`.

## <a name='tcpdump'></a>Configure Analytics and Firewall

TCP Dump allows us to do some checks on the live stream when it is broadcasting which will allow troubleshooting if the need arise

- install tcpdump to see the operation of the livestream  
    `sudo apt-get install tcpdump`  
    `sudo tcpdump -i any port 1935`
    `ctl` + `c` to exit the stream
    
- open firewall port 1935 on lightsail it is closed, this is done in networking on the instance
    
- open firewall port 433 on lightsail it is closed, this is done in networking on the instance

## <a name='viewer'></a>Build Your Viewer

Now, let's set up your front page lobby so that surfers can visit your website and listen to or watch your live stream

- paste this code to create the viewer page and open it  
    `sudo nano /var/www/html/stream.html`
    
- copy this code and paste it in the nano editor

```
<!DOCTYPE html>
<html>
<head>
    <title>My Streaming Server || Better Call Jason</title>
    <meta name="author" content="Jason Nobles">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@1/css/pico.min.css">
    <style>

        @media only screen and (min-width: 992px) {
            article {
                width: 800px;
                margin: auto;
            }
            video {
                width: 700px;
                height: 450px;
            }
            audio {
                width: 700px;
            }

        }

        body > main {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: calc(100vh - 7rem);
            padding: 1em;
        }

        article {
            overflow: hidden;
            text-align: center;
        }


        @media only screen and (max-width: 768px) {

            video {
                width: 100%;
                height: auto;
            }
            audio {
                width: 100%;
            }
        }


    </style>
</head>

<body>

<main>

    <nav>
        <ul>
            <li></li>
        </ul>
        <ul>
            <li><strong>bcj.one/live</strong></li>
        </ul>
        <ul>
            <li></li>
        </ul>
    </nav>

    <article>
        <div>
            <hgroup>
                <h1>My Streaming Server</h1>
                <h2>Video</h2>
            </hgroup>
            <link href="https://vjs.zencdn.net/8.5.2/video-js.css" rel="stylesheet" />
            <script src="https://vjs.zencdn.net/8.5.2/video.min.js"></script>
            <div data-vjs-player>
                <video id="my_video_1"
                       class="video-js vjs-default-skin vjs-fluid vjs-16-9 vjs-big-play-centered"
                       controls
                       preload='auto'
                       data-setup='{}'>
                    <source src="https://live.bcj.one/hls/streamkey.m3u8" type="application/x-mpegURL">
                </video>
            </div>
        </div>
        <br>
        <br>
        <div>
            <hgroup>

                <h2>Audio</h2>
            </hgroup>
            <div data-vjs-player>
                <audio
                        id="my_audio_1"
                        class="video-js vjs-default-skin vjs-fluid"
                        controls
                        preload="auto"
                >
                    <source src="https://live.bcj.one/hls/streamkey.m3u8" type="application/x-mpegURL">
                </audio>
            </div>

        </div>
    </article>
</main>

<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
<script src="https://cdn.jsdelivr.net/npm/videojs-contrib-hls.js@latest"></script>
<script>
    var player = videojs('my_video_1');
    player.play();

    var player2 = videojs('my_audio_1');
    player2.play();
</script>
<script>
    const themeApplier = {
        // Config
        darkScheme: "dark",
        rootAttribute: "data-theme",

        // Init
        init() {
            this.applyScheme();
        },

        // Apply dark scheme
        applyScheme() {
            document
                .querySelector("html")
                .setAttribute(this.rootAttribute, this.darkScheme);
        },
    };

    // Init
    themeApplier.init();
</script>

</body>

</html>
```
- after pasting click `ctl` + `s` and then `ctl` + `x`<br />

- Now create your livestream and try it out!

## <a name='Donate'></a>Donate

If you found this project helpful and would like to contribute to its ongoing development, consider making a donation.

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](YOUR_PAYPAL_LINK)

Thanks for your support!
