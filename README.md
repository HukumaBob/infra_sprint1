# Kittygram is a social network for sharing photos of your pets

## Project Description
Users can register, upload photos of their cats with a brief description, and view photos of other users' cats. It is a fully functional project consisting of a Django backend application and a React frontend application. The main goal is to learn how to deploy the project to a server.

## Technologies

- Python 3.9
- Django==3.2.3
- djangorestframework==3.12.4
- Nginx
- Gunicorn

## Installing the Project on a Local Computer from the Repository
- Clone the repository: `git clone <your_repository_address>`
- Navigate to the cloned repository directory.
- Create a virtual environment: `python3 -m venv venv`
- Install the dependencies: `pip install -r requirements.txt`
- Create a `.env` file in the `/backend/kittygram_backend/` directory.
- Add your `SECRET_KEY` to the `.env` file in the following format: `SECRET_KEY = '<your_key>'`

# Deploying the Project to a Remote Server

## Connecting the Server to Your GitHub Account
- If Git is not installed, install it with the command `sudo apt install git`.
- Generate SSH key pair on the server using `ssh-keygen` command.
- Save the public key to your GitHub account. Display the key in the terminal using `cat .ssh/id_rsa.pub`. Copy the key from `ssh-rsa` to the end. Add this key to your GitHub account.
- Clone the project from GitHub to the server: `git clone git@github.com:Your_Account/<Project_Name>.git`

## Running the Backend on the Server
- Install the package manager and the utility for creating a virtual environment: `sudo apt install python3-pip python3-venv -y`
- In the project directory, create and activate a virtual environment: `python3 -m venv venv` and `source venv/bin/activate`
- Install the dependencies: `pip install -r requirements.txt`
- Perform migrations: `python manage.py migrate`
- Create a superuser: `python manage.py createsuperuser`
- Edit `settings.py` on the server: Add the external IP address of your server and the addresses `127.0.0.1` and `localhost` to the `ALLOWED_HOSTS` list. `ALLOWED_HOSTS = ['<your_server_external_address>', '127.0.0.1', 'localhost']`

## Running the Frontend on the Server
- Install Node.js on the server with the following commands:
  ```
  curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
  sudo apt-get install -y nodejs
  ```
- Install the frontend dependencies. From the `<your_project>/frontend/` directory, run `npm i`.

## Installing and Running Gunicorn
- Activate the project's virtual environment and install the Gunicorn package: `pip install gunicorn==20.1.0`
- Open the project's `settings.py` file and set the `DEBUG` constant to `False`: `DEBUG = False`
- Create a `gunicorn.service` file in the `/etc/systemd/system/` directory using a text editor (`sudo nano /etc/systemd/system/gunicorn_kittygram.service`) with the following content (without comments):

  ```
  [Unit]
  Description=gunicorn daemon
  After=network.target

  [Service]
  User=<username>
  WorkingDirectory=/home/<your_system_username>/<your_project_name>/backend/
  ExecStart=/home/<your_system_username>/<your_project_name>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

  [Install]
  WantedBy=multi-user.target
  ```

  To find the path to Gunicorn, you can use the command `which gunicorn` while the virtual environment is activated.

## Installing and Configuring Nginx

- On the server, execute the command from any directory: `sudo apt install nginx -y`
- To set restrictions on open ports, execute the following commands one by one: `sudo ufw allow 'Nginx Full'`  `sudo ufw allow OpenSSH`
- Enable the firewall: `sudo ufw enable`

### Building and Deploying Frontend Static Files
- Navigate to the `<your_project>/frontend/` directory and run the command `npm run build`. The result will be saved in the `/frontend/build/` directory. Copy the contents of the `/frontend/build/` directory to the system directory `/var/www/`.
- Open the Nginx configuration file with the command `sudo nano /etc/nginx/sites-enabled/default` and replace its contents with the following code:

  ```
  server {
      listen 80;
      server_name <your_server_public_ip>;

      location / {
          root   /var/www/<your_project_name>;
          index  index.html index.htm;
          try_files $uri /index.html;
      }
  }
  ```

- Save the changes, check the configuration for correctness with `sudo nginx -t`, and reload the Nginx configuration: `sudo systemctl reload nginx`

### Configuring Proxy Pass for Backend Requests
- Open the Nginx configuration file `/etc/nginx/sites-enabled/default` and add the following `location` block:

  ```
  server {
      listen 80;
      server_name <your_server_public_ip>;

      location /api/ {
          proxy_pass http://127.0.0.1:8080;
      }

      location /admin/ {
          proxy_pass http://127.0.0.1:8080;
      }

      location / {
          root   /var/www/<your_project_name>;
          index  index.html index.htm;
          try_files $uri /index.html;
      }
  }
  ```

- Save the changes, check the configuration for correctness with `sudo nginx -t`, and reload the Nginx configuration: `sudo systemctl reload nginx`

### Building and Configuring Backend Static Files and Media Files
- In the `settings.py` file of your backend project, set the following settings:

  ```
  MEDIA_URL = '/media/'
  MEDIA_ROOT = os.path.join(BASE_DIR, 'var', 'www', 'kittygram', 'media')
  STATIC_URL = 'static_backend'
  STATIC_ROOT = BASE_DIR / 'static_backend'
  ```

- Activate the virtual environment, navigate to the directory containing the `manage.py` file, and run the command `python manage.py collectstatic`.
- A `static_backend/` directory will be created in the `<your_project>/backend/` directory.
- Create directory `/var/www/kittygram/media`
- Use this command:
  ```
  sudo chown -R <user_name> /var/www/kittygram/media/
  ```
- In default file add:
```
    location /media/ {
    alias /var/www/kittygram/media/;
    }
```
- Copy the `static_backend/` directory to `/var/www/<your_project_name>/`.
- Copy the `/var/www/kittygram/media` directory to `/var/www/<your_project_name>/`.

## Adding Domain Name to Django Settings
- In the `settings.py` file, add your domain name to the `ALLOWED_HOSTS` list:

  ```
  ALLOWED_HOSTS = ['your_server_ip', '127.0.0.1', 'localhost', 'your_domain']
  ```

- Save the changes and restart Gunicorn: `sudo systemctl restart gunicorn`
- Update the Nginx configuration. Open the configuration file with the command `sudo nano /etc/nginx/sites-enabled/default`.
- Add your domain name (without `< >`) to the `server_name` line, separated by a space:

  ```
  server {
  ...
      server_name <your_ip> <your_domain>;
  ...
  }
  ```

- Check the configuration with `sudo nginx -t` and reload it with `sudo systemctl reload nginx` to apply the changes.

## Obtaining and Configuring an SSL Certificate
**Installing Certbot**
- Log in to your server and execute the following commands:

  ```
  sudo apt install snapd
  sudo snap install core; sudo snap refresh core
  sudo snap install --classic certbot
  sudo ln -s /snap/bin/certbot /usr/bin/certbot
  ```

- Run Certbot and obtain an SSL certificate:

  ```
  sudo certbot --nginx
  ```

- The certificate will be automatically saved on your server in the system directory `/etc/ssl/`. The Nginx configuration file `/etc/nginx/sites-enabled/default` will also be automatically updated with the new settings and paths to the certificate.
- Reload the Nginx configuration: `sudo systemctl reload nginx`
