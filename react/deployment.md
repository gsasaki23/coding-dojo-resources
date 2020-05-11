# For the Following Folder Structure (Before `npm run build`)
    ├─ project-name/
    | ├─ client/
    | | ├─ node_modules/
    | | ├─ public/
    | | ├─ src/
    | | ├─ package.json
    | ├─ server/
    | | ├─ config/
    | | ├─ controllers/
    | | ├─ node_modules/
    | | ├─ models/
    | | ├─ routes/
    | | ├─ package.json
    | | ├─ server.js
    | - .gitignore (with only node_modules/)

## Assumptions
1. The default `create-react-app` git related files are deleted
2. There is already a working repo

## Github Steps
1. Open terminal to `client` folder
2. `npm run build`
3. Push to master
4. Double check that the `.gitignore` worked, and no `node_modules` folders are present

---
## AWS EC2 Virtual Machine setup (***!!!!!!!REPLACE {{text}} WITH APPROPRIATE TEXT - DON'T KEEP CURLY BRACES!!!!!!!***)
[AWS Management Console](https://console.aws.amazon.com/ec2/v2/home)

# Instance Setup
1. `Launch a virtual machine With EC2`
2. Step 1: Select `Ubuntu Server 18.04 LTS (HVM) 64-bit`
3. Step 2: Select `t2.micro`, `Next`
4. Click `Next:` until Step 6
5. Step 6: For type `SSH`, change Source to `My IP`
6. Step 6: `Add Rule` for type `HTTP` with Source `Anywhere`
7. Step 6: `Add Rule` for type `HTTPS` with Source `Anywhere`, then `Review and Launch`
8. Reuse or create new PEM key. Save it to a folder you will **NEVER** use on GitHub
9. Check the box, `Launch Instances`
10. Coffee Break

# Ubuntu Setup
11. Find the instance you just created. Change Name to whatever is appropriate
12. Select and click `Connect`
13. Go to PEM file location, open terminal (BASH for PC Users)
13. Copy + Paste `chmod` line from AWS into terminal
14. Copy + Paste `ssh -i` line from AWS into terminal, `yes` if prompted
    - If connection time out error:
      1. description tab below click: Security groups launch-wizard-2
      2. Inbound tab
      3. click edit
      4. add SSH with source My IP (any time your ip address changes you need to do this)
15. Update Ubuntu software with `sudo apt-get update`
16. Install dependencies with `sudo apt install nodejs npm nginx git -y`
17. Node version will be 8.10.0, which is old. We use a PPA (personal package archive) to get a newer version.
    ```
    nodejs -v # should print 8.10.0
    curl -sL https://deb.nodesource.com/setup_10.x -o nodesource_setup.sh
    sudo bash nodesource_setup.sh
    sudo apt install nodejs
    nodejs -v # should now print 10.19.0
    sudo apt install build-essential
    ```
18. Go to the IP shown in AWS, check that `Welcome to nginx!` page is showing
19. Clone GitHub repo with `git clone {{YOUR_REPO_URL}}`
20. Set repo name to variable with `export repoName={{YOUR_REPO_NAME}}`
21. Check with `echo $repoName`

# Front-End
22. Move to client folder, replace the default nginx folder with the production react app
    - `cd ~/$repoName/client`
    - `sudo rm -rf /var/www/html`
    - `sudo mv build /var/www/html`
    - `sudo service nginx restart`
23. Check the IP to make sure that the project front-end is now showing instead of default nginx
24. If working with a Back-end, overwrite the `localhost` routes 
    - Uses grep to find all lines with string `localhost` and replace using Stream Editor
    ```
    sudo grep -rl localhost /var/www/html | xargs sed -i 's/http:\/\/localhost:8000//g'
    ```
25. `sudo service nginx restart` and check IP

# Back-End
26. Move to server folder with `cd ~/$repoName/server`
26. Revive the `node_modules` we gitignored with whatever is listed in `packages.json`: `npm i`
27. Install MongoDB
    - `wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -`
    - `echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list`
    - `sudo apt update`
    - `sudo apt install -y mongodb-org`
28. Start MongoDB daemon and check status    
    - `sudo service mongod start`
    - `service mongod status`
    - If a green message shows, OK to `ctrl+C` out
29. Overwrite nginx settings so that requests starting with `/api` are pointed to the back-end, and everything else to `index.html`
    - * Assuming that the port used was 8000. If not, edit it
    - `sudo rm /etc/nginx/sites-available/default`
    - `sudo vim /etc/nginx/sites-available/default`
    - `i` to enter insert mode, paste the rest AFTER FIXING SERVER NAME
    ```
    # {{YOUR_REPO_NAME}} Configuration {{MM-DD-YYYY}}
    server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
        server_name {{YOUR_REPO_NAME}};
        location /api {
            proxy_pass http://localhost:8000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;    
        }
        location / {
            try_files $uri $uri/ =404;
        }
        error_page 404 /index.html;
    }
    ```
    - `esc` then `:wq` `enter` to save
30. Resets and check IP
    - `cd ~/$repoName/server`
    - `sudo service nginx restart`
    - `node server.js`
31. Install pm2 as a process manager to keep back-end running
    - `sudo npm i pm2 -g`
    - `pm2 start server.js`
    - `pm2 status` to see status

32. Ready!

# To update deployed project with code changes
1. Push changes to GitHub repo
2. cd to repo folder on the AWS server: `cd ~/$repoName`
3. `git pull`
4. Restart nginx with `sudo service nginx restart` and pm2