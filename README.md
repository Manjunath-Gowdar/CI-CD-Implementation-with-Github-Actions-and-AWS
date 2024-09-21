# CI-CD-with-Github-Actions-and-AWS

Implementing CI / CD using Github Actions and AWS


# setting up ec2 instace at AWS

- launch ec2 instance with aws, with ubuntu os,
- enable hhtp, https, and also enable 'inbound rule in ec2 security group' to
  enable port 5000 (temporary enable).
- create static ip address in 'Elastic IP Address' of aws and associate the
  static ip address with the instance you have created.
- login to ec2, and install the necessay tools, run the following command.

  ```
  sudo apt update && sudo apt upgrade -y

  sudo apt install -y nodejs npm git nginx rsync

  sudo npm install -g pm2

  git clone https://github.com/your-username/your-repository.git

  "npm run build" @ root, if script of package.json contains build as ("build": "npm install && npm install --prefix frontend && npm run build --prefix frontend")
  ```

- ## replace the git clone url with your github url
- ## if build script in empty add this at the end of script as ("build": "npm install && npm install --prefix frontend && npm run build --prefix frontend")

---

---

# Domain name

- in this example i have used hostgator to purchase domain name 'picfeel.com'
  and i have created the subdomain as 'cherryshades.picfeel.com' using CNAME of
  hostgator, the CNAME 'cherryshades.picfeel.com' points to Public Ipv4 DNS of
  aws, eg-'ec2-100-29-80-144.compute-1.amazonaws.com'.
- you can get ssl certificates(https) to only domain name, if you just have ip
  address you will not get ssl certificates.

---

---

# Setup Nginx

- go to the path /etc/nginx/sites-available.
- edit the file default, and replace the code with below code, using sudo and
  nano
- the below code will make nginx to point the 'localhost:5000' to
  'cherryshades.picfeel.com' all the other url like ipv4 address(100.29.80.144),
  and ipv4 DNS(ec2-100-29-80-144.compute-1.amazonaws.com), point to 404 error on
  web page.

```
server {
    listen 80;
    server_name cherryshades.picfeel.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name _;

    location / {
        return 404;
    }
}

```

- ## replace cherryshades.picfeel.com to your url

- test the nginx systex, with below code
  ```
  sudo nginx -t
  ```
- enable the configuration (-sf flag will force overwrite the existing file,) if
  any error remove only 'f' from the below '-sf' flag
  ```
  sudo ln -sf /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
  ```
- restart the nginx
  ```
  sudo systemctl restart nginx
  ```

---

---

# setting up ssl to get (https), secure encryption,

- go to the path 'home/ubuntu/Cherryshades'

  ```
  sudo apt install certbot python3-certbot-nginx

  sudo certbot --nginx -d cherryshades.picfeel.com
  ```

- ## replace 'cherryshades.picfeel.com' with your domain name
- this will create additionally code in nginx's default file, which looks like
  below

  ```
      server {
        server_name cherryshades.picfeel.com;

        location / {
            proxy_pass http://localhost:5000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/cherryshades.picfeel.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/cherryshades.picfeel.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    }

    server {
        listen 80;
        server_name _;

        location / {
            return 404;
        }
    }

    server {
        if ($host = cherryshades.picfeel.com) {
            return 301 https://$host$request_uri;
        } # managed by Certbot


        listen 80;
        server_name cherryshades.picfeel.com;
        return 404; # managed by Certbot


    }

  ```

---

---

# start pm2

- pm2 will keep on the repl of nginx on forever(even if terminal is cloned),
  even if we clone the terminol.
  ```
  pm2 start /home/ubuntu/app-name/backend/server.js --name pm2-repl-name
  ```
- ## replace app-name to your app name(root folder), replace pm2-repl-name to your choice of name
- if the path of server.js is in /root/backend/server.js, use the above code
  else change the path to your chosen path.
- ## The below command will make pm2 start automatically on ubuntu startup

```
  pm2 startup

  sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u ubuntu --hp /home/ubuntu

  pm2 save
```

- ## Basic pm2 commands

  ```
    # Start an application with PM2
    pm2 start app.js --name my-app

    # List all PM2 processes
    pm2 list

    # Restart an application
    pm2 restart my-app

    # Stop an application
    pm2 stop my-app

    # Delete an application from PM2
    pm2 delete my-app

    # Monitor logs for all applications managed by PM2
    pm2 logs

    # View logs for a specific application
    pm2 logs my-app

    # Save the current process list
    pm2 save

    # Auto-start PM2 on system startup
    pm2 startup

    # Kill PM2 daemon
    pm2 kill
  ```

---

---

# Setting up Github Actions

- in the root folder create '.github' folder, inside that create 'workflows'
  folder, inside that create a file called 'deploy.yml', the path looks like
  root(appname)/.github/workflows/deploy.yml.
- Github Actions will get triggered on github event like git push to main
  branch.(there is lot of github events).
- Github actions has its own environment to execute the code
- ***

  ***

  ***

  ***

  ***

  ***

  ***

  ***

  ***

  ***

  ***

# Extras

## The file structure of my MERN stack web application.

```
cherryshades/
├── backend/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   ├── orderController.js
│   │   ├── productController.js
│   │   └── userController.js
│   ├── data/
│   │   ├── products.js
│   │   └── users.js
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   └── errorMiddleware.js
│   ├── models/
│   │   ├── orderModel.js
│   │   ├── productModel.js
│   │   └── userModel.js
│   ├── routes/
│   │   ├── orderRoutes.js
│   │   ├── productRoutes.js
│   │   ├── uploadRoutes.js
│   │   └── userRoutes.js
│   ├── utils/
│   │   └── generateToken.js
│   ├── seeder.js
│   ├── server.js
├── frontend/
│   ├── build/
│   │   ├── images/
│   │   │   ├── banaPowder.png
│   │   │   ├── blush.png
│   │   │   ├── boldLiner.png
│   │   │   ├── eyeLash.png
│   │   │   └── ... (other image files)
│   │   ├── static/
│   │   │   ├── css/
│   │   │   │   └── ... (CSS files)
│   │   │   ├── js/
│   │   │   │   └── ... (JS files)
│   │   └── ... (other build files)
│   ├── public/
│   │   ├── images/
│   │   │   ├── anaPowder.png
│   │   │   ├── blush.png
│   │   │   ├── boldLiner.png
│   │   │   ├── eyeLash.png
│   │   │   └── ... (other image files)
│   │   ├── favicon.ico
│   │   ├── index.html
│   │   ├── logo192.png
│   │   ├── manifest.json
│   │   └── robots.txt
│   ├── src/
│   │   ├── actions/
│   │   │   ├── cartAction.js
│   │   │   ├── orderAction.js
│   │   │   ├── productAction.js
│   │   │   └── userAction.js
│   │   ├── components/
│   │   │   ├── footer.js
│   │   │   └── header.js
│   │   ├── constants/
│   │   │   ├── cartConstants.js
│   │   │   └── orderConstants.js
│   │   ├── reducers/
│   │   │   ├── cartReducer.js
│   │   │   └── orderReducer.js
│   │   ├── screens/
│   │   │   ├── homeScreen.js
│   │   │   ├── loginScreen.js
│   │   │   ├── orderScreen.js
│   │   │   └── paymentScreen.js
│   │   ├── App.js
│   │   ├── bootstrap.css
│   │   ├── index.css
│   │   ├── index.js
│   │   ├── products.js
│   │   └── store.js
│   ├── package.json
│   ├── package-lock.json
│   ├── README.md
├── upload/
│   └── image.png
├── .env
├── .gitignore
├── package.json
├── package-lock.json
```

---

# pm2 basic commands

```
#To start the application.
pm2 start server.js --name cherryshades-pm2

#To check status
pm2 status

# Start an application
pm2 start app.js

# List all running applications
pm2 list

# Restart an application
pm2 restart app_name_or_id

# Stop an application
pm2 stop app_name_or_id

# Delete an application from PM2 process list
pm2 delete app_name_or_id

# Show application logs
pm2 logs app_name_or_id

# Monitor resource usage of processes managed by PM2
pm2 monit

# Save the current process list
pm2 save

# Resurrect processes from the last saved list
pm2 resurrect

# Display version of PM2
pm2 -v

# Show detailed information about a process
pm2 show app_name_or_id

```

---

---

# nginx basics

```
  Start Nginx:
  sudo systemctl start nginx

  Stop Nginx:
  sudo systemctl stop nginx

  Restart Nginx:
  sudo systemctl restart nginx

  Reload Nginx configuration (without restarting):
  sudo systemctl reload nginx

  Check Nginx configuration syntax:
  sudo nginx -t

  Verify the Symlink of nginx:
  ls -l /etc/nginx/sites-enabled/

  View Nginx version:
  nginx -v

  Check Nginx status:
  sudo systemctl status nginx

  Accessing Logs
  sudo tail -f /var/log/nginx/access.log

  Error Logs
  sudo tail -f /var/log/nginx/error.log

  - use 'tail -f' to get logs in Real-Time
  - use 'less' for detailed viewing, perss 'q' to exit less repl
  - example sudo less /var/log/nginx/access.log
            sudo less /var/log/nginx/error.log

  Start Nginx at boot:
  sudo systemctl enable nginx

  Stop Nginx from starting at boot:
  sudo systemctl disable nginx
```

# Basics of nano - ubuntu text editor

- to copy and save the file in nano text editor,
- sudo nano default (to enter/edit the file default)
- ctrl+shift+c to copy the code,
- ctrl+shift+v to paste the selected text,
- ctrl+o - to save the file,
- ctrl+x - to exit the file,
- "cd -" to go to previous folder/directory

---

---

---

---

# IMPORTANT

## Note

- sometimes even if everything is correct, pm2 or nodejs throws 'mongo-uri =
  undefiend' error, may be its because of Doman name confuguration of CNAME, or
  something else, please wait for 6 to 24 hours, and log back into ec2 and run
  the application
