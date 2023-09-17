# Own Your Stream --- Toolkit For Online Freedom Episode 1

Follow the steps below to set up your own streaming server. 

![alt text](https://github.com/Better-Call-Jason/nginx-streaming-server-app/blob/main/episode1-thumbnail2.png?raw=true)

## Table of Contents

1. [About](#about)
2. [Getting Started](#Prerequisites)
3. [Set Up](#Installation)
4. [Build Your Receiver](#receiver)
5. [Build Your Broadcaster](#broadcaster)
6. [Add SSL](#ssl)
7. [Configure OBS](#obs)
8. [Configure Analytics and Firewall](#tcpdump)
9. [Build Your Viewer](#viewer)
10. [Go Live](#final)
11. [Keep Your Stream Secure](#secure)
12. [Donate](#donate)
13. [Parting Shot](#goodbye)

## <a name='about'></a>About

Hello, it is Jason. Thanks for dropping by. As a developer, I have directly seen the power that big tech has over our lives. In a very small way, I'd like to push back against that trend and dissent against their power over us. I am tired of seeing voices marginalized and cancelled by big tech at the behest of governments and corporate actors simply for having an opinion contrary to the experts and authorities. I think the world can handle a little disagreement and a little mis-information. Follow these steps below and you will be unstoppable by big tech.

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
    - Hate cloud computers? This entire project can exist on a Rasberry PI 4B <br />
- point your domain to your created instance's ip
    - navigate to your domain dns
    - create an A record
    - enter a subdomain or leave blank or @
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
```
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
      - FYI you will need your stream key to be used in building the viewer
    
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

Now, let's set up your front page lobby so that surfers can visit your website and listen to or watch your live stream. I designed the front end code to be stylish and responsive, functional on both desktop and mobile browsers. There's a lot going on so I won't waste your time to discuss it. Big thanks to PicoCSS for amazing semantic CSS which is what is used to style the front end. Also using VideoJS, a free video player that handles HLS streams. Don't worry if you don't understand what I just said, just copy and paste and you'll have an amazing lobby for your surfers to visit.

- You'll Need `YOUR_STREAM_KEY` and `YOUR_DOMAIN_NAME`<br />
    - only replace the key and domain but keep the `/hls/` and `.m3u8`<br />
      `https://YOUR_DOMAIN_NAME/hls/YOUR_STREAM_KEY.m3u8`

- paste this code to create the viewer page and open it  
    `sudo nano /var/www/html/stream.html`
    
- copy this code and paste it in the nano editor, make the changes to add your stream key and domain name. It has to be done twice. One for the audio and one for the video.

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
                    <source src="https://YOUR_DOMAIN_NAME/hls/YOUR_STREAM_KEY.m3u8" type="application/x-mpegURL">
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
                    <source src="https://YOUR_DOMAIN_NAME/hls/YOUR_STREAM_KEY.m3u8" type="application/x-mpegURL">
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

## <a name='final'></a>Go Live

The last thing to do is to test the configurations. If the response says ok, then you are all set. If you run into an issue look at the errors and try to figure it out. If you need help, please reach out. Most times, errors at this stage are typos or code syntax errors. Enter the two commands below

`sudo nginx -t`  <br />

if everything is good and the status is okay then :<br />

`sudo systemctl restart nginx` <br />

This restarts the service and you are now ready to broadcast your live stream. 

- for troubleshooting your stream use tcpdump to see the operation of the livestream. If you are receiving a stream to the server the command below will be overwhelmed with data, if just an empty screen then something is wrong with your set up.  
      
    `sudo tcpdump -i any port 1935`<br />
  
  press  `ctl` + `c` to exit the stream

## <a name='secure'></a>Secure Your Stream

Prevent anyone else from broadcasting to your server by making the following changes. Only deploy these changes once you have fully tested and determined the system is working as intended. 

- configure nginx to receive only the stream from your ip address. Get your real IP and enter it in the provided code. You cannot broadcast from behind a VPN. The entire streaming server is a VPN on its own.<br /> 
    ```sudo nano /etc/nginx/nginx.conf```  
- copy and insert this code below with the changes at the bottom of the page. Make sure to replace `YOUR_LOCAL_IP_ADDRESS` with your real IP address<br />

```
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;
            allow publish YOUR_LOCAL_IP_ADDRESS;
            deny publish all;

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

- now one more time let's test our configs:
  
`sudo nginx -t`  <br />

if everything is good and the status is okay then :<br />

`sudo systemctl restart nginx` <br />

This restarts the service and you are now ready to broadcast your live stream and you are the only one that can stream to your server now. 


## <a name='donate'></a>Donate

This project was months in development. Once I had a working model that I tested repeatedly I spent weeks in tutorial preparation trying to make it as easy as possible for a novice or an expert to own their own live stream server. While I make it all look very easy to do, there are a lot of complex things involved here. If my tutorial has helped you out, please email me at everydayjason@protonmail.com and tell me about it. I can't wait to hear about your successes! And, please consider making a BTC or LTC donation below.


[![Donate Crypto](https://nowpayments.io/images/embeds/donation-button-white.svg "Donate Button")](https://nowpayments.io/donation?api_key=3JB4N4H-57MME28-GXQF9CK-G61YRXR&source=lk_donation&medium=referral)


Thanks for your support!

## <a name='goodbye'></a>Parting Shot

And remember as James Evan Pilato says: <br /> 
`Don't Hate The Media, Become The Media!`
