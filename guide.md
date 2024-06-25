# guide for vps assignment

1. create Digital Ocean & Squarespace accounts
2. configure droplet
   1. check `~/.ssh` file to see if there are any existing ssh keys and make sure you don't need them for any other uses (like github!)
   2. create a new key with `ssh-keygen`, then get the public key with `cat ~/.ssh/id_rsa.pub` and paste it the Droplet config
3. open iTerm and ssh into created Ubuntu droplet with `ssh root@DO_public_ip`
4. create non-root user with system admin permissions `adduser new_user` -> `usermod -aG sudo new_user`
5. move the droplet's ssh setup files into our new user `rsync --archive --chown=new_user:new_user ~/.ssh /home/new_user` and then reconnect to ubuntu droplet using `ssh new_user@DO_public_ip`
6. setup initial firewall settings with `ufw` and add `OpenSSH` with `sudo ufw allow OpenSSH` and `sudo ufw enable`
7. remove root user login access by inverting the value for "PermitRootLogin" `/etc/ssh/sshd_config` and then restart the ssh service: `sudo service ssh restart`
8. install `nodejs`, `npm`, and `express`
9. setup another ssh with `ssh-keygen` ON THE DROPLET INSTANCE and (???)modify the `/home/new_user/.ssh/config` file so the host is `github.com`
10. clone static demo app from github via ssh (you need to get the public key's info which you setup in step 9 and save it on github)
11. buy a domain (i used squarespace)
12. add the domain name on Digital Ocean (/networking/domains) and add two new records for `www` and `@` for a total of 5 DNS records (three of which are provided by DO)
13. navigate to your domain listed in squarespace's "Domains" page and go to th DNS settings, make sure the default squarespace dns settings are removed then go to 'Domain Nameservers' copy and paste in the three Digital Ocean name servers from before
14. log back into droplet with your created user and install nginx on droplet `sudo apt install nginx`
15. modify firewall settings with `sudo ufw allow 'Nginx Full'`
16. check that nginx is active with `systemctl status nginx`, the resultant message should show an active nginx configuration
17. input: and ensure that the `server_name` entry has both `www.yourdomain.com` & `yourdomain.com` on it and that the `location / {}` entry looks like this:

```
location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}
```

18. verify that the configuration file we just modified doesn't have syntax errors and appears to be working as intended with: `sudo nginx -t`
19. reload nginx `sudo systemctl reload nginx`
20. install `sudo npm install pm2@latest -g` (a process manager to run our app when our Droplet ubuntu server starts up)
21. configure what process the process manager should initiate at startup... `pm2 start file_which_starts_our_expressapp.js` then fetch the startup command with: `pm2 startup systemd` and copy&paste the command and save changes `pm2 save`
22. reboot the instance with `sudo reboot` (this will end the connection to our droplet)
23. go and check our domain out in browser to verify things are working
