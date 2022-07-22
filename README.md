# Create and deploy Django app with Postgres, Nginx and Gunicorn

#### Description:

- Launch an EC2 server.
- Install the dependencies to run the Django app.
- Use Postgres as a database.
- Create a Django app and deploy it on the server.

#### Step-1: Launch an EC2 server

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

#### Step-2: Install the dependencies to run the Django app.

Firstly, we need to update local apt package index using ```sudo apt update```

![14](https://user-images.githubusercontent.com/74168188/180445474-da5a7513-143c-41ad-b138-0ff9d6e383c6.png)
![15](https://user-images.githubusercontent.com/74168188/180445503-e7e9e8c1-25d0-442b-a865-daf524d29341.png)

Then we need to install some packages i.e. python3-venv, python3-dev, libpq-dev, postgresql, postgresql-contrib, nginx, and curl.
```sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl```

![16](https://user-images.githubusercontent.com/74168188/180448999-9bd261f4-a587-421b-9f65-530c1c7c2bcc.png)
![17](https://user-images.githubusercontent.com/74168188/180449320-39180b78-fee1-475c-b09d-b3daf35968f5.png)
