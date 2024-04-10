# Setting up Firewall, Reverse Proxy, and a Backend

## Setting up Firewall

Install the firewall with:

`sudo pacman -S ufw`

### Configuring the firewall

By default, ufw is configured to:

- Deny all incoming traffic
- Allow all outgoing traffic

First we'll allow SSH so we can connect to the machine:

`sudo ufw allow ssh`

Because we're setting up a web server reverse proxy, we'll also allow HTTP:

`sudo ufw allow http`

Let's start the firewall service:

`sudo systemctl enable --now ufw`

To list the allowed connections, use

`sudo ufw status verbose`

And enable the firewall

`sudo ufw enable`

Our firewall is ready! Let's move on to the backend setup

## Backend Setup

Download the hello-server file and place it in `/web/backend`

Create a unit file, `hello-server.service` in `/etc/systemd/system`

```bash
[Unit]
Description=Hello Service

[Service]
Type=simple
ExecStart=/web/backend/hello-service

[Install]
WantedBy=multi-user.target
```

Refresh systemctl:

`sudo systemctl daemon-reload`

Start the backend server:

`sudo systemctl start --now hello-server.service`

## Setup Reverse Proxy

Edit your /etc/nginx/sites-available/nginx-2420.conf to add the new `proxy_pass` configuration

```bash
server {
    listen 80;
    listen [::]:80;
    root /web/html/nginx-2420;
    location / {
        index index.php index.html index.htm;
    }
    location /hey {
	    proxy_pass http://127.0.0.1:8080;
    }
    location /echo {
    	    proxy_pass http://127.0.0.1:8080;
    }
}
```

Restart nginx

`sudo systemctl restart nginx`

## Testing

`curl http://<IP of droplet>/hey`

Should receive:

> Hey there

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"message": "Hello from your server"}\n' \
  http://<IP of droplet>/echo
```

Should receive:

`{"message": "Hello from your server"}`

And that's all! 🚀
