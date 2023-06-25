# Create and deploy Django app with Postgres, Nginx and Gunicorn

#### Description:

• Launch an EC2 instance.
• Install the necessary dependencies to run the Django application.
• Utilize Postgres as the database.
• Create a Django application and deploy it on the instance.

#### Step-1: Launch an EC2 instance.

Firstly, log in to the AWS management console.

![Img-1](https://user-images.githubusercontent.com/74168188/178555843-f062573f-166c-4b06-b947-d2d11da46507.png)

After login, navigate to the search bar, type EC2, and select EC2

![Img-2](https://user-images.githubusercontent.com/74168188/178555883-5e169bfd-d205-4b99-9992-8dca7e957ff2.png)

Now, the EC2 dashboard will appear. Click Instances

![Img-3](https://user-images.githubusercontent.com/74168188/178555921-6213742a-726c-4532-923e-3edc4c4d1413.png)

Click the Launch instances button.

![Img-4](https://user-images.githubusercontent.com/74168188/178555939-529e74f0-6ece-43dc-aa1c-98d6b7f2deda.png)

To launch an EC2 instance, a few details are required, i.e., instance name, OS image (AMI), instance type, etc.

![5](https://user-images.githubusercontent.com/74168188/180404364-0f02ce97-b70e-49a7-8728-05e31ddeda7e.png)

![Img-6](https://user-images.githubusercontent.com/74168188/178556050-f90b180a-0dca-48fb-b30b-8365f9ac8f28.png)
![Img-7](https://user-images.githubusercontent.com/74168188/178556073-b0a18233-2e38-4dbd-926d-f7e411f7fa06.png)

Select a key pair to attach with the instance to log in with that key. If you do not have key pair, then create a new key pair by clicking Create a new key pair. Moreover, according to usage, download key pair in .pem or .ppk format.

![8](https://user-images.githubusercontent.com/74168188/179713553-227871ba-db62-496f-b361-2cf1ef3ff50e.png)

Now, to allow traffic from the internet, we need to create a rule in network settings.

![9](https://user-images.githubusercontent.com/74168188/179716719-a040568f-8b89-43df-817c-f39b35b19337.png)

We are good to go to launch an instance for this. Click the Launch instance button. If everything is correctly set up, you will find a success message on the screen. Click on the instance id to navigate to the EC2 dashboard.

![Img-11](https://user-images.githubusercontent.com/74168188/178556301-ac2e5bdd-7efa-4ad4-99d9-b669e599eefd.png)

Now, take the public IP from the EC2 dashboard and use it to login inside the instance using ssh. First move to the directory where the key is downloaded. In my case it's in downloads directory.

![12](https://user-images.githubusercontent.com/74168188/180442775-2d40c558-e255-47c1-a5e9-de47bc588b76.png)
![13](https://user-images.githubusercontent.com/74168188/180442952-271d82fd-06ea-450b-bfda-96cd109e7c88.png)

Finally, it will be logged in successfully if everything is configured correctly.

#### Step-2: Install the necessary dependencies to run the Django application.

Firstly, we need to update local apt package index using ```sudo apt update```

![14](https://user-images.githubusercontent.com/74168188/180445474-da5a7513-143c-41ad-b138-0ff9d6e383c6.png)
![15](https://user-images.githubusercontent.com/74168188/180445503-e7e9e8c1-25d0-442b-a865-daf524d29341.png)

Then we need to install some packages i.e. **python3-venv** to create virtual environments, **python3-dev** will give python development files needed to build Gunicorn, **libpq-dev** will give libraries to interact with postgres, **postgresql** for postgres database, **postgresql-contrib**, **nginx** for nginx server, and **curl**. To install all packages use ```sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl```

![16](https://user-images.githubusercontent.com/74168188/180448999-9bd261f4-a587-421b-9f65-530c1c7c2bcc.png)
![17](https://user-images.githubusercontent.com/74168188/180449320-39180b78-fee1-475c-b09d-b3daf35968f5.png)

#### Step-3: Utilize Postgres as the database.

To check postgreSQL version use ```psql -V```

![IMG](https://user-images.githubusercontent.com/74168188/180759453-c9aec058-0c8a-4a55-99c2-d56ed83fcc53.png)

Now time to create postgreSQL database and user.
For login inside postgres use ```sudo -u postgres psql```

![image](https://user-images.githubusercontent.com/74168188/180630607-8960ef62-46f2-4e43-aa11-fffde96f3f4e.png)

Now we have to create a database for django project using ```create database <DATABASE_NAME>;```

![image](https://user-images.githubusercontent.com/74168188/180630850-6661a0b0-07c0-452d-b23d-748d94578848.png)

Next, create a database user for our project with secure password using ```create user <DB_USER_NAME> with password '<PASSWORD>';```

![image](https://user-images.githubusercontent.com/74168188/180631021-f32c57f4-5286-488a-b22d-284f15096712.png)

Now we have to modify few connection parameters for the user created to speedup database operations. So, set the default character encoding to **UTF-8**, which Django expects. Then, set the default transaction isolation scheme to **read committed**, which blocks reads from uncommitted transactions. Lastly, set the timezone to **UTC**. Set all these using :
```
alter role <DB_USER_NAME> set client_encoding to 'utf-8';
alter role <DB_USER_NAME> set default_transaction_isolation to 'read comitted';
alter role <DB_USER_NAME> set timezone to 'utc';
```

![image](https://user-images.githubusercontent.com/74168188/180631324-5d3959f5-f0a9-4af4-b045-941e32ae0203.png)

Now, we can give the new user access to administer the new database using ```grant all privileges on database <DATABASE_NAME> to <DB_USER_NAME>;```

![image](https://user-images.githubusercontent.com/74168188/180631578-d7c53fc7-d61c-47ac-80e7-2dae2186216c.png)

Postgres is now set up so that Django can connect to and manage its database information. Now exit out of postgreSQL prompt by typing ```\q```

![image](https://user-images.githubusercontent.com/74168188/180631950-64957b32-8d87-45d2-af01-55a421c8cb84.png)

#### Step-4: Create a Django application and deploy it on the instance.

Firstly, create and change into a directory to keep all project files:
```
mkdir mydjangoproject
cd mydjangoproject
```

![image](https://user-images.githubusercontent.com/74168188/180632092-ebb54cd6-c13d-4bf1-85e5-d6ff6c6493a3.png)

Now we have to create a  python virtual environment inside this directory using ```python3 -m venv djangoenv```

![image](https://user-images.githubusercontent.com/74168188/180632213-c1726f42-51da-48b4-9e96-2e0d60acfe30.png)

Before installing project's Python requirements, we need to activate the virtual environment using ```source djangoenv/bin/activate```

![image](https://user-images.githubusercontent.com/74168188/180632270-b06e0a53-6093-4082-8600-dc128b4931e9.png)

Now install Django, Gunicorn and psycopg2 PostgreSQL adapter using ```pip install django gunicorn psycopg2-binary```

![image](https://user-images.githubusercontent.com/74168188/180632471-cd4aa8ef-3d8d-445d-b6d1-80846827e9a9.png)

Now create a django project in the project directory using ```django-admin startproject djangoapp .```

![image](https://user-images.githubusercontent.com/74168188/180632815-84048b2b-c2ad-443c-af0a-0e424bad8bab.png)

**Django Project Structure:**
Django uses a directory structure to arrange the different parts of the web application.
1. manage.py
    This file contains code for runserver, or makemigration, or migration.

2. _init_.py
    This is an empty file and is only present to tell this directory is a package.
    
3. settings.py
    This is main file of Django project and stores information of databases and allowed hosts.
    
4. urls.py
    This file handles all the URLs of our web application.
    
5. wsgi.py
    This file mainly concerns with the WSGI server and is used for deploying our applications on to servers.
    
6. asgi.py
    ASGI can be considered as a succeeder interface to the WSGI.

We have to make changes in setting.py file. In **ALLOWED_HOSTS** section we have to write the server's address or domain names may be used to connect to Django app.

![image](https://user-images.githubusercontent.com/74168188/180633085-418ca9aa-a50a-468f-a04b-c9e1f97402c1.png)
![image](https://user-images.githubusercontent.com/74168188/180634552-5805c632-8908-431f-9469-a22b76c3e177.png)

Next in **DATABASES** section change the setting with postgreSQL database information. We need to tell Django to use the psycopg2 adapter. Give database name, the database username, the database user’s password, and then specify that the database is located on the local computer.

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'mydb',
        'USER': 'mydbuserr',
        'PASSWORD': 'django',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

![image](https://user-images.githubusercontent.com/74168188/180634303-6ceec4ab-ad59-4715-866c-b76e70a795a8.png)

At last of this file we need to add below lines which tells where the static files should be placed. This is necessary so that Nginx can handle request for these items.
```
import os
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```

![image](https://user-images.githubusercontent.com/74168188/180634007-d70ac0d1-c577-4841-b16b-41aae204397b.png)

Now save and exit by pressing **Esc** then type ```:wq!```

Now, we can migrate the initial database schema to our PostgreSQL database using the management script:

```
~/mydjangoproject/manage.py makemigrations
~/mydjangoproject/manage.py migrate
```

![image](https://user-images.githubusercontent.com/74168188/180635076-526b8d1a-a139-4143-bf36-c872b5706427.png)

Create an administrative user for the project by ```~/mydjangoproject/manage.py createsuperuser``` it will prompt to enter username, email and password.

![image](https://user-images.githubusercontent.com/74168188/180635126-756dedfe-b364-4ddf-b229-66ed95a79c11.png)

We can collect all of the static content into the directory location that configured using ```~mydjangoproject/manage.py collectstatic```

![image](https://user-images.githubusercontent.com/74168188/180640420-d2615465-f9a9-4419-88bc-44fbc6470ae9.png)

Finally, test out our project by starting up the Django development server using ```~/mydjangoproject/manage.py runserver 0.0.0.0:8000```

![image](https://user-images.githubusercontent.com/74168188/180634647-d577459e-7214-40ce-8ee0-0515fed63b4b.png)

After this create a firewall rule to allow port 8000 then in web browser visit **EC2_PUBLIC_IP:8000**

![image](https://user-images.githubusercontent.com/74168188/180634659-5eb24796-94c1-4cff-8929-28838b057b4f.png)
![image](https://user-images.githubusercontent.com/74168188/180634766-7030ec26-9517-4155-b859-5ce53b4809bf.png)

If we append /admin to the end of EC2_PUBLIC_IP:8000 then admin page will appear

![image](https://user-images.githubusercontent.com/74168188/180635215-f09aa449-ff53-4195-b77a-0461c2c2775c.png)

After authenticating, we can access the default Django admin interface:

![image](https://user-images.githubusercontent.com/74168188/180635293-ae94b9b6-8365-420c-a824-8087819915f0.png)

To close development server hit **CTRL+C**

Now to test Gunicorn can server the application enter into **mydjangoproject** directory and run Gunicorn using:
```
cd ~/mydjangoproject
gunicorn --bind 0.0.0.0:8000 djangoapp.wsgi
```
![image](https://user-images.githubusercontent.com/74168188/180635504-9c7c1b6a-5c86-4dd4-b47b-db46b89ae61c.png)

![image](https://user-images.githubusercontent.com/74168188/180635498-e925794c-c73e-4308-b00d-a8b28defc20d.png)

Now hit **CTRL+C** to stop Gunicorn. And back out of virtual environment using ```deactivate```

![image](https://user-images.githubusercontent.com/74168188/180635556-cd82c03f-411a-491d-900d-19f5f68bf45a.png)

Now we have to create systemd socket and service file for Gunicorn

For socket file:
```
sudo vim /etc/systemd/system/gunicorn.socket
```

![image](https://user-images.githubusercontent.com/74168188/180635716-864aef23-642b-4366-9617-f60c83cb7faa.png)

and inside socket file create a [Unit] section to describe the socket, a [Socket] section to define the socket location, and an [Install] section to make sure the socket is created at the right time:

```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```
![image](https://user-images.githubusercontent.com/74168188/180635858-1bf5dd75-1d59-4c7b-80e4-58f5c1d079d2.png)

For service file:
```
sudo vim /etc/systemd/system/gunicorn.service
```

![image](https://user-images.githubusercontent.com/74168188/180636007-28483bd9-ac15-404f-be64-efeaf6a472c0.png)

```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/mydjangoproject
ExecStart=/home/ubuntu/mydjangoproject/djangoenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          djangoapp.wsgi:application

[Install]
WantedBy=multi-user.target
```

![image](https://user-images.githubusercontent.com/74168188/180637525-c4798e89-ad47-4785-8e95-a4d4d5206ecb.png)

Now start and enable the Gunicorn socket. This will create the socket file at /run/gunicorn.sock now and at boot. When a connection is made to that socket, systemd will automatically start the gunicorn.service to handle it.

```
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

![image](https://user-images.githubusercontent.com/74168188/180636261-88bcf87e-1d43-4309-9601-37ce75d56ec4.png)

To check the status of gunicorn.socket use ```sudo systemctl status gunicorn.socket```

![image](https://user-images.githubusercontent.com/74168188/180636298-2f68b73d-eea2-46da-a82d-56a37b273a6e.png)

We only started the gunicorn.socket unit, the gunicorn.service will not be active yet since the socket has not yet received any connections. To check ```sudo systemctl status gunicorn```

![image](https://user-images.githubusercontent.com/74168188/180636417-a050c4e5-399b-4cfe-9b88-0a64353502b7.png)

To active gunicorn service hit to socket using ```curl --unix-socket /run/gunicorn.sock localhost```

![image](https://user-images.githubusercontent.com/74168188/180640134-9fa1406c-4622-4d8a-8d4e-e7d3b18ff71c.png)

Last step is to configure Nginx to proxy pass to Gunicorn. For this we have to create a server block file.
```
sudo vim /etc/nginx/sites-available/django
```
![image](https://user-images.githubusercontent.com/74168188/180640510-cbdf6d2c-073b-4017-8571-24c9264aface.png)

```
server {
    listen 80;
    server_name 15.206.70.93;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/sammy/myprojectdir;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

![image](https://user-images.githubusercontent.com/74168188/180640576-34394e31-0c39-4769-9818-5fa5f28c51c1.png)

We can enable this file by linking it with **sites-enabled** directory using ```sudo ln -s /etc/nginx/sites-available/django /etc/nginx/sites-enabled/```

![image](https://user-images.githubusercontent.com/74168188/180640713-55f5ae33-712b-4363-8b5f-27ea06bcd6bf.png)

To check syntax error of Nginx configuration use ```sudo nginx -t```

![image](https://user-images.githubusercontent.com/74168188/180640769-7d78cf18-58e6-4aea-8dbc-a061357607ea.png)

Now time to restart nginx using ```sudo systemctl restart nginx``` but make sure to create a firewall rule for port 80.

![image](https://user-images.githubusercontent.com/74168188/180641291-1a5507e6-1a04-4471-be3f-ffb4541fd2c4.png)

![image](https://user-images.githubusercontent.com/74168188/180640896-4eca3908-6515-44fb-8dc3-e36c58cf16cd.png)

Let's hit **EC2_PUBLIC_IP**

![image](https://user-images.githubusercontent.com/74168188/180641317-e548ce15-3486-4aed-97f9-a14da818309f.png)

As you can see we get the result.

### Thank You
