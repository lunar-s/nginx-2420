# How to setup nginx on an Arch DigitalOcean Droplet

## What you'll do

- Install a text editor
- Install nginx
- Configure and run nginx

### Installing Vim (text editor) and nginx

0. Upgrade your system if it's been over a week!

   `sudo pacman -Syu`

1. Install a text editor (Vim is this example)

   `sudo pacman -S vim`

2. Make sure Vim is installed

   `vim --version | grep "VIM - Vi"`

> Note: Use grep here to shorten the `vim --version`, it's quite lengthy!

3. Install nginx

   `sudo pacman -S nginx`

4. Check its status

   `sudo systemctl status nginx`

   It's not running and it's disabled!

5. Start nginx

   `sudo systemctl start nginx`

6. Enable it to start automatically on boot

   `sudo systemctl enable nginx`

> Note: if you want to condense this into a single command `sudo systemctl start --now nginx`

You should see a reply:

> Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service

This means in `/etc/systemd/system/multi-user.target.wants` there is a new service (also known as a unit file).
This unit file is in charge of handling the nginx binary.
It is not required (it's wanted) so the system can start without it.

In status, it is now enabled and active!

```
nginx.service - A high performance web server and a reverse proxy server
  Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
  Active: active (running) since Wed 2024-03-20 18:51:36 UTC; 1h 25min ago
```

All right, with that out of the way let's set up our nginx site!

### Creating the site

1. Create a folder in the root directory that will contain your site files. For the rest of the tutorial, `nginx-2420` can be named anything you want. Just be consistent with how you name and locate this file!

`sudo mkdir -p /web/html/nginx-2420`

Sudo is needed because this is being created in the root folder, the `-p` flag is to create nested directories in a single command.

2. Copy and paste the following HTML page into a file named `index.html` inside the folder you just created

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>2420</title>
    <style>
      * {
        color: #db4b4b;
        background: #16161e;
      }
      body {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100vh;
        margin: 0;
      }
      h1 {
        text-align: center;
        font-family: sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>All your base are belong to us</h1>
  </body>
</html>
```

### Configuring nginx

In nginx's configuration folder, `/etc/nginx`, create two directories.

`sudo mkdir /etc/nginx/sites-available` and `sudo mkdir /etc/nginx/sites-enabled`

In the `sites-available` directory, create the file `nginx-2420.conf` with the following code:

```
server {
    listen 80;
    listen [::]:80;
    root /web/html/nginx-2420;
    location / {
        index index.php index.html index.htm;
    }
}
```

This is a server block. Each server block connects a path to nginx's service on the specified port.

> Note: This means the port will listen on port 80 (the default HTTP port), and serve our `index.html` from the location `/web/html/nginx-2420`

In `/etc/nginx/nginx.conf`, add `include sites-enabled/*;` to your http block like so:

```
http {
    include mime.types;
    default_type application/octet-stream;
    include sites-enabled/*;
    ...
}
```

To enable a site, create a symlink of the conf to sites-enabled

`ln -s /etc/nginx/sites-available/nginx-2420.conf /etc/nginx/sites-enabled/nginx-2420.conf`

To disable a site, unlink the symlink!

`unlink /etc/nginx/sites-enabled/example.conf`

By doing it this way, you can easily manage which sites you want to enable or disable. This is better than having everything in the `nginx.conf` file, which can be changed by your package manager or accidentally deleted when uninstalling nginx.

Because the `nginx.conf` file was edited, restart the nginx service:

`sudo systemctl restart nginx`

### Finishing up

Because nginx comes with a default site, you must delete that server block.

In the `nginx.conf` file, delete the first server block within the main http block.

---

With the `nginx.conf` file complete, symlinks in place, and index.html specified, you can now visit your site!

In your browser, type in your Droplet's IP and see it in action 🚀

### Troubleshooting

Having errors when launching your nginx service? Use `sudo nginx -t` for a better explanation on what is wrong!

Usually, these errors are formatting mistakes in `nginx.conf` or `nginx-2420.conf`.
