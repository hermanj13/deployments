# AWS Deployment
## Part 1: Get prepared
#### Core Setup Requirements
* An AWS account
* A GitHub or bitbucket account
* An internet connection
* Git should be installed locally
#### Get going
* Create a local, functional project
* Keep that version, version controlled via git - good practice anyway.
#### Make a GitHub/bitbucket repository
* Push your project to that GitHub/bitbucket repository
## Part 2: Set up AWS
* Enter AWS, and click launch new instance.
* Select `Ubuntu 16.04 LTS`
* Select `t2.micro`
* Set security settings:
* `ssh 0.0.0.0, (Anywhere or myIP)`
* `http 0.0.0.0 (Anywhere)`
* `https 0.0.0.0 (Anywhere, or don't set it)`
* Download a `.pem` key from AWS
* Move the `.pem` file to an appropriate folder on your system
* Change user permission on your `.pem`  file using the command: `chmod 400 {{mypem}}.pem`
#### For PC, that command might require kitty or putty or bash terminal
We are now ready to enter the cloud server!

## Part 3: Enter cloud server
#### Navigate to the folder where your .pem file is!

(you can use the ‘connect’ button on Amazon AWS to get the next line of code)

#### For PC, the command below might require kitty or putty or bash terminal
* `ssh -i {{mypem}}.pem ubuntu@{{yourAWS.ip}}` again you can get this exact line of code from Connect  on AWS. (The command is found below example).
* In the ubuntu terminal: These establish some basic dependencies for deployment and the Linux server.

```bash
> sudo apt-get update
> sudo apt-get install -y build-essential openssl libssl-dev pkg-config
```

* In the ubuntu terminal, one at a time because they require confirmation: (these install basic node and npm)

```bash
> sudo apt-get install -y nodejs nodejs-legacy
```

Note: In case this does not work, try sudo apt install nodejs-legacy instead.

```bash
> sudo apt-get install npm
> sudo npm cache clean -f
```
**(The cache clean -f, forcibly cleans the cache.  This will give an interesting comment:))**

* In the ubuntu terminal: These install the node package manager n and updated node.

```bash
> sudo npm install -g n
> sudo n stable (or whichever node version you want e.g. 5.9.0)
```

* `node -v` should give you the stable version of node, or the version that you just installed.

* Install NGINX and git:

```bash
> sudo apt-get install nginx
> sudo apt-get install git
```

Make your file folder:

```bash
> sudo mkdir /var/www
```

Note: You may want to check first if you already have the file folder. If it's already existing, proceed to the next step.

* Enter the folder:
```bash
> cd /var/www
```
* Clone your project:
```bash
> sudo git clone {{your project file path on github/bitbucket}}
```
##### At this point, you should be able to change directories into your project and run your server. It will most likely fail, because of not having mongod up and running, but running the project should be as simple as node server.js or a similar command like npm start.

## Part 4: Set up NGINX
* Go to nginx’s sites-available directory :
```bash
> cd /etc/nginx/sites-available
```
* Enter vim:
```bash
> sudo vim {{project_name}}
```
##### vim is a terminal-based text editor for more info see: vim-adventures.com/ or other vim learning tools. The key commands for us are i which allows us to type, esc which turns off insert and then after esc :wq which says write and quit.
* Paste and modify the following code into vim after hitting i:
```bash
server {
    listen 80;
    location / {
        proxy_pass http://{{PRIVATE-IP}}:{{NODE-PROJECT-PORT}};
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

This code says: have the reverse proxy server (nginx) listen at port 80. When going to root /, listen for http requests as though you were actually http://<your private ip> and the port your server is listening e.g @8000 or @6789 etc.

Learn more from nginx: http://nginx.org/en/docs/http/ngx_http_proxy_module.html

* Remove the defaults from /etc/nginx/sites-available
```bash
> sudo rm default
```

* Create a symbolic link from sites-enabled to sites available:
```bash
> sudo ln -s /etc/nginx/sites-available/{{project_name}} /etc/nginx/sites-enabled/{{project_name}}
```

* Remove the defaults from /etc/nginx/sites-enabled/

```bash
> cd /etc/nginx/sites-enabled/
> sudo rm default
```

## Part 5: Project Dependencies and PM2
* Install pm2 globally (https://www.npmjs.com/package/pm2.5) (https://www.npmjs.com/package/pm2). This is a production process manager that allows us to run node processes in the background.
```bash
> sudo npm install pm2 -g
```

* Try some stuff with pm2!

```bash
> cd /var/www/{{project_name}}
> pm2 start server.js
> pm2 stop 0
> pm2 restart 0
> sudo service nginx reload && sudo service nginx restart
```

##### Probably not quite working yet but close

* You might have some components that you still need to install: (get your dependencies from npm (assuming your git project has a package.json))

```bash
> sudo npm install
```
* IF USING BOWER (assuming you have a bower.json)
```bash
> sudo npm install bower -g
> sudo bower install --allow-root
```

## Part 6: Mongodb
* The last thing, setting up mongodb!
* Set up a key
```bash
> sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```

* Setup mongodb in a source list  Ubuntu Docs. Pick the version you are currently using from the below snippets:

### Version 12.04

```bash
> echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu precise/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

### Version 14.04

```bash
> echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
### Version 16.04

```bash
> echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
##### Important:That is all one line!



* Re-update to integrate mongo
```bash
> sudo apt-get update
```
* install mongo
```bash
> sudo apt-get install -y mongodb-org
-or-
> sudo apt-get install -y mongodb (if the above does not work)
```

* Start mongo (probably already started)

```bash
> sudo service mongod start
```

If you're having trouble getting mongod to start with `service`, try running it manually, as below:

```bash
> sudo mongod
```

Check for an error code. It might ask you to build a `/data/db` folder. If you can get your DB server to start like this, we can then pass it to pm2 as to not completely take up our terminal.

```bash
> sudo pm2 start mongod
```

* Restart your pm2 project and make sure the nginx config’s are working:
```bash
> pm2 stop 0
> pm2 restart 0
> sudo service nginx reload && sudo service nginx restart
```

* At this point, the nginx commands should have shown 2 OKs and you should be off and running. Go to the AWS public IP and see your site live!
