# Bridge Node Setup
In this guide we will set up start a Warthog node with websocket support to support incoming connections from browser nodes. 

Due to browser restrictions, TLS encryption is required on the browser side. To address this issue we will need to use a reverse proxy for websocket connections that is hosted a public server that and pointed to by a domain name.
This reverse proxy will forward websocket traffic to the node and perform the TLS encryption.

!!!
A bridge node should accept incoming connections from the public internet. It is recommended to use a virtual or dedicated server for this purpose.
!!!

# Setting up the node
We can either use the precompiled binary, compile from scratch or use the Dockerfile. The only relevant difference from starting a node normally is to add two command line flags `--ws-x-forwarded-for` and `--ws-port=10001` (or any other port you like to use).

## Using the precompiled binary
Nodes of version 0.7.x and higher support the bridge feature. Download an appropriate release from [here](https://github.com/warthog-network/Warthog/releases).

In order to support incoming websocket connections from browser nodes we need to start the node with the `--ws-x-forwarded-for` and `--ws-port=10001` flags. 

## Get the Warthog Node from the `network_refactor` branch
Right now only the `network_refactor` branch supports bridge nodes so you need to check out this branch: 

Firstly, you need to clone the Warthog core repository:
```
git clone https://github.com/warthog-network/Warthog.git
```
Then enter the repository directory and switch to the `network_refactor` branch ```git switch network_refactor```.

We can either compile directly from source or within a docker container. In this guide we demonstrate the latter.

We first build the docker image: 
```
docker build . -f dockerfiles/build_linux -t warthog
```

We can now run the container with the `--ws-x-forwarded-for` and `--ws-port=10001` flags and also map the websocket port `-p 10001:10001`:
```shell
docker run -it --mount src="$HOME"/.warthog/,target=/warthog/.warthog/,type=bind -p 10001:10001 -p 3000:3000 warthog --ws-x-forwarded-for --ws-port=10001
```

# Setting up the nginx
Nginx can be installed with the command
```console
$ sudo apt install nginx -y
```
## Getting a domain name
You can use your own domain name or contact us to create an A or AAA record for a subdomain of the form `node<x>.warthog.network`.

## Configuring nginx
In `/etc/nginx/sites-enabled/default` enable reverse proxying. For example:
```
upstream warthog_node {
  server 127.0.0.1:10001 max_fails=5 fail_timeout=60s;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name node2.warthog.network;
    return 301 https://$host$request_uri;
}
# server {
# 	listen 443 ssl default_server;
# 	listen [::]:443 ssl default_server;
# 	ssl_certificate     /etc/letsencrypt/live/node2.warthog.network/fullchain.pem;
# 	ssl_certificate_key /etc/letsencrypt/live/node2.warthog.network/privkey.pem;
# 
# 	server_name node2.warthog.network;
# 
#     location /ws {
#             rewrite /ws/(.*) /$1  break;
#             #proxy_set_header X-Real-IP $remote_addr;
#             #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#             proxy_set_header Host $host;
#             proxy_pass http://warthog_node;
#             proxy_http_version 1.1;
#             proxy_set_header Upgrade $http_upgrade;
#             proxy_set_header Connection "upgrade";
#     }
# }
```

 Note that some part of the configuration is commented out because we do not have the `ssl_certificate` and `ssl_certificate_key` files yet. We will uncomment that part later when we have these files (see next step).

When uncommented, this configuration sets up a virtual server for the host `node2.warthog.network`. Websocket connections to `node2.warthog.network/ws` are proxied to the node which should listen on localhost port 10001. 

## Obtaining a certificate
Either get a certificate from your domain registrar and adapt the certificate paths accordingly or get a free certificate using certbot.

Type the following command:
```
$ sudo certbot certonly --nginx --register-unsafely-without-email
```


!!!
If you see this error
```console
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Could not choose appropriate plugin: The requested nginx plugin does not appear to be installed
The requested nginx plugin does not appear to be installed
```

you need to install the nginx certbot plugin:
```console
$ sudo apt-get install python3-certbot-nginx
```
!!!

You should see this output:
```
certbot certonly --nginx --register-unsafely-without-email
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Registering without email!

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: node2.warthog.network
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for node2.warthog.network
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/node2.warthog.network/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/node2.warthog.network/privkey.pem
   Your cert will expire on 2024-11-25. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

The output above lists the paths of the `fullchain.pem` and the `privkey.pem` files. Make sure you use these paths in the configuration file.

Now you can uncomment the `server` block of the nginx configuration file above and restart nginx again with
```
systemctl restart nginx
```

When your node is running with the `--ws-x-forwarded-for` and `--ws-port=10001` flags you should be able to accept websocket connections on `<your domain>/ws` from browser nodes. 

You now have successfully set up a bridge node which connects browser nodes with the normal nodes. Thank you! Please inform us about your node and domain such that we can include your node in the list of bridge nodes.
