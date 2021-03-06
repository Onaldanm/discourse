**Set up Discourse on a cloud server in under 30 minutes** with zero knowledge of Ruby/Rails or Linux shell using our [Discourse Docker image][dd]. We prefer [Digital Ocean][do], although these steps may work on other cloud providers that also support Docker. Let's begin!

# Create New Digital Ocean Droplet

[Sign up for Digital Ocean][do], update billing info, then begin creating your new cloud server (Droplet).

Discourse requires a minimum of 1 GB RAM for small communities; we recommend 2 GB RAM for medium communities. We'll use "discourse" as the Hostname.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3506/a6b550bd2b05b76b.png" width="638" height="500"> 

Install Discourse on Ubuntu 12.04.4 LTS x64. We always recommend using [the current LTS distribution][lts].

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3399/f3fc67ee6aa90ea4.png" width="690" height="477"> 

You will receive a mail from Digital Ocean with the root password to your Droplet. (However, if you know [how to use SSH keys](https://www.google.com/search?q=digitalocean+ssh+keys), you may not need a password to log in.)

# Access Your Droplet

Connect to your Droplet via SSH:

    ssh root@192.168.1.1

(Alternately, use [Putty][put] on Windows)

Replace `192.168.1.1` with the IP address of your Droplet.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/2999/0934a0158459ec3f.png" width="571" height="130"> 

You will be asked for permission to connect, type `yes`, then enter the root password, which is in the email Digital Ocean sent you when the Droplet was set up.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3000/8209c1e40c9d70a8.png" width="570" height="278"> 

# Update the kernel

    apt-get update
    apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3001/e94722e882f28994.png" width="566" height="339"> 

Reboot the server:

    reboot

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3003/d3cc759ced335d25.png" width="532" height="155"> 

This will log you out from your SSH session, so reconnect:

    ssh root@192.168.1.1

# Set up Swap (if needed)

- If you're using the minimum 1 GB install, you *must* [set up a swap file](https://meta.discourse.org/t/create-a-swapfile-for-your-linux-server/13880) of 1 GB minimum. 

- If you're using 2 GB+ memory, you can probably get by without a swap file.

# Install Git

    apt-get install git

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3002/eafbf14df8eee832.png" width="572" height="263"> 

# Install Docker

    wget -qO- https://get.docker.io/ | sh

# Install Discourse

Create a `/var/docker` folder where all the Docker related stuff will reside:

    mkdir /var/docker

Clone the [Official Discourse Docker Image][dd] into this `/var/docker` folder:

    git clone https://github.com/discourse/discourse_docker.git /var/docker

Switch to your Docker folder:

    cd /var/docker

Copy the `samples/standalone.yml` file into the `containers` folder as `app.yml`, so the path becomes `containers/app.yml`:

    cp samples/standalone.yml containers/app.yml

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3005/5c253f4657e2133f.png" width="571" height="56"> 

# Edit Discourse Configuration

Edit `app.yml`:

    nano containers/app.yml

(We recommend Nano because it works like a typical GUI text editor, just use your arrow keys. Hit <kbd>Ctrl</kbd><kbd>O</kbd> then <kbd>Enter</kbd> to save and <kbd>Ctrl</kbd><kbd>X</kbd> to exit. However, feel free to choose whatever text editor you like.)

- Set `DISCOURSE_DEVELOPER_EMAILS` and `DISCOURSE_HOSTNAME`. 

- If you are using a 1 GB instance, set `UNICORN_WORKERS` to 2 so you have more memory room.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/2979/e6fedbde9b471880.png" width="565" height="172"> 

If you set `DISCOURSE_HOSTNAME` to `discourse.example.com`, this means you want your Discourse available at `http://discourse.example.com/`. You'll need to update the DNS A record for this domain with the IP address of your server.

**Email is critical to notifications in Discourse. If you do not configure email before bootstrapping you will have a broken site!**

- If you already have a mail server, put your existing mail server credentials in the `app.yml` file.

- Otherwise, create a free account on [**Mandrill**][man] (or [Mailgun][gun], or [Mailjet][jet]), and put your mail credentials (available via the Mandrill dashboard) in the `app.yml` file. The settings you want to change are `DISCOURSE_SMTP_ADDRESS`, `DISCOURSE_SMTP_PORT`, `DISCOURSE_SMTP_USER_NAME`, `DISCOURSE_SMTP_PASSWORD`.

- Be sure you remove the comment character and space `# ` from the beginning of these mail configuration lines!

- Don't forget to set the [SPF and DKIM records](http://help.mandrill.com/entries/21751322-What-are-SPF-and-DKIM-and-do-I-need-to-set-them-up-) up for your domain name. In Mandrill, that's under Sending Domains, View DKIM/SPF setup instructions. Also, the name of your droplet is your reverse PTR record; rename your droplet to match your domain name `discourse.example.com`.

# Bootstrap Discourse

Be sure to save the `app.yml` file, and begin bootstrapping Discourse:

    ./launcher bootstrap app

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3007/c0596ad3d330ae71.png" width="567" height="138"> 

This command can take up to 8 minutes. It is automagically configuring your Discourse environment.

After that completes, start Discourse:

    ./launcher start app

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3008/ced00cf4782f020c.png" width="568" height="137"> 

Congratulations! You now have your own instance of Discourse, accessible via the domain name you entered in `app.yml` earlier.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3507/d01eee7415f860f2.png" width="690" height="291">

You can also access it by visiting the server IP address directly, e.g. `http://192.168.1.1`.

# Log In and Become Admin

Sign into your Discourse instance. There should be a reminder visible on the site about which email was used for the  `DISCOURSE_DEVELOPER_EMAILS` address. Be sure you log in with that email, and your account will be made Admin by default.

# Post-Install Maintenance

To **upgrade Discourse to the latest version**, visit `/admin/docker`, refresh the page a few times (yes, seriously) and then press the Upgrade button at the top. View the live output at the bottom of your browser to see when things are complete. You should see:


    Killed sidekiq
    Restarting unicorn pid: 37


Then you know it's complete. (Yes, we will be improving this process soon!)

The `launcher` command in the `/var/docker` folder can be used for various kinds of maintenance:

```
Usage: launcher COMMAND CONFIG
Commands:
    start:      Start/initialize a container
    stop:       Stop a running container
    restart:    Restart a container
    destroy:    Stop and remove a container
    ssh:        Start a bash shell in a running container
    logs:       Docker logs for container
    bootstrap:  Bootstrap a container for the config based on a template
    rebuild:    Rebuild a container (destroy old, bootstrap, start new)
```

# Other Awesome Stuff

Do you want...

- Users to log in via Facebook? [Configure Facebook logins](https://meta.discourse.org/t/configuring-facebook-login-for-discourse/13394).

- Users to log in via Twitter? [Configure Twitter logins](https://meta.discourse.org/t/configuring-twitter-login-for-discourse/13395/last).

- Users to post reples via email? [Configure reply via email](https://meta.discourse.org/t/set-up-reply-via-email-support/14003).

- Automatic daily backups? [Configure backups](https://meta.discourse.org/t/hot-off-the-presses-automated-backup-support/13805).
 
- HTTPS / SSL support? [Configure SSL](https://meta.discourse.org/t/allowing-ssl-for-your-discourse-docker-setup/13847).
 
- Host multiple Discourse sites on the same server? [Configure multisite](https://meta.discourse.org/t/multisite-configuration-with-docker/14084).

If anything needs to be improved in this guide, feel free to ask on [meta.discourse.org][meta], or even better, submit a pull request.

   [dd]: https://github.com/discourse/discourse_docker
  [man]: https://mandrillapp.com
  [ssh]: https://help.github.com/articles/generating-ssh-keys
 [meta]: https://meta.discourse.org/t/beginners-guide-to-deploy-discourse-on-digital-ocean-using-docker/12156
   [do]: https://www.digitalocean.com/?refcode=5fa48ac82415
  [lts]: https://wiki.ubuntu.com/LTS
  [jet]: http://www.mailjet.com/pricing
  [gun]: http://www.mailgun.com/
  [put]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
