# Video
EMBED VIDEO HERE

# Video Notes
Welcome everyone, today we're going to be setting up our own JupyterHub server.
The instance we're going to spin up today is intended to host a small number
of users, up to about a hundred. If you need to have more users than that, say
for a large class, then you can either spin up multiple of these servers or
use Kubernetes. 

1. Pre-requisites for this Video
    * Have a DigitalOcean Account
    * Have a domain purchased (if you desire https with LetsEncrypt)
    * DNS can be setup through DO, or through your own provider


2. Create a DigitalOcean Droplet
    * Must be Ubuntu 18.04
    * We're gonna go with the 4GB / 2 CPUs droplet
        * At least 768MB RAM for TLJH to install
    * Use an SSH Key or One Time Password to login, your choice. 
    * For sizing your instances, remember
        * Recommended Memory = (Max concurrent users * Max mem per user) + 128MB
        * Recommended CPU = (Max concurrent users * Max CPU usage per user) + 20%
        * Disk space = (Total users * Max disk usage per user) + 2GB
3. Create an A record in DNS for your server. This will ensure we get a
certificate from LetsEncrypt and will allow us to navigate to it later.
4. SSH to your droplet and setup proper user accounts. You can follow the 
tutorial [here](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04).
5. Run this code on your machine. This will setup the JupyterHub Server
```bash
#!/bin/bash
wget https://raw.githubusercontent.com/jupyterhub/the-littlest-jupyterhub/master/bootstrap/bootstrap.py \
sudo python3 boostrap.py --admin mason
```
6. Setup HTTPS for your JupyterHub
```bash
sudo tljh-config set https.enabled true
sudo tljh-config set https.letsencrypt.email mason@digitalocean.com
sudo tljh-config add-item https.letsencrypt.domains jupyterhub.egger.codes
```
View the config and confirm your changes are correct.
    ```bash
    sudo tljh-config show
    ```
Reload the proxy when you're ready
    ```bash
    sudo tljh-config reload proxy
    ```

    ???+ warning "Potential Issues"
        * If your Jupyterhub fails to launch a server when you login simply
        `sudo systemctl restart jupyterhub.service` and it should work.
        * I've also had issues where the certificate wasn't working properly. Reloading
        either the proxy or the `jupyterhub.service` fixed this issue.

7. Install package from terminal in the JupyterHub UI. Only Admins can install 
packages.
```bash
sudo -E conda install -c conda-forge numpy
sudo -E pip install flask
```
8. Navigate to you JupyterHub via the DNS name you setup and login with the
username you specified earlier. 

??? info "Bonus: Use CloudInit to setup"
    All in one installation cloudinit. This can be put into the `User Data` section
    when spinning up your droplet.
    ```
    #!/bin/bash
    curl https://raw.githubusercontent.com/jupyterhub/the-littlest-jupyterhub/master/bootstrap/bootstrap.py \
    | sudo python3 - \
        --admin mason
    tljh-config set https.enabled true
    tljh-config set https.letsencrypt.email megger@digitalocean.com
    tljh-config add-item https.letsencrypt.domains test1.egger.codes
    tljh-config reload proxy
    systemctl restart jupyterhub.service
    ```

# Extra Resources
[Original JupyterHub Tutorial](https://the-littlest-jupyterhub.readthedocs.io/en/latest/install/digitalocean.html)  
[JupyterHub Documentation](https://the-littlest-jupyterhub.readthedocs.io/en/latest)