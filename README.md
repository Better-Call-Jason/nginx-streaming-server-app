# Own Your Stream --- Toolkit For Online Freedom

Follow the steps below to set up your own streaming server. 

![alt text](https://github.com/Better-Call-Jason/nginx-streaming-server-app/blob/main/episode1-thumbnail2.png?raw=true)

## Table of Contents

1. [Getting Started](#Prerequisites)
2. [Installation](#Installation)
3. [Code Examples](#Code)
4. [Donate](#Donate)

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
  
## <a name='Installation'></a>Installation

To install this project, follow these steps...

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
