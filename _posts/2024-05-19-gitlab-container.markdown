---
layout: post
title:  "Gitlab container"
date:   2024-05-19 21:27:07 -0400
categories: gitlab infrastructure scm
---
Often times customers will want to see what a workflow will look like in their environment. Sometimes it’s also a benefit being able to run the system locally so that you don’t run into the limits of GitLab’s SaaS free plan.

For this use case it will probably be SCM, CI/CD, or Broker integrations that the customer wants to see. 

### Step 1 - Official docs ###

Take a look at the official GitLab Docker images documentation, https://docs.gitlab.com/ee/install/docker.html.

### Step 2 - Setup Directories ###

Create a directory where you will store the GitLab containers config, data, and logs directories.

```
mkdir ~/gitlab
export GITLAB_HOME=$HOME/gitlab
```

### Step 3 - Run the Container ###

The only variables you may need to change in the command below is the hostname. You can technically set that to whatever you want locally as long as you don’t need to access it from the Internet.

```
docker run --detach \
  --hostname gitlab.perciballi.ca \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'https://gitlab.perciballi.ca'" \
  --publish 443:443 --publish 80:80 --publish 222:22 \
  --name gitlab \
  --restart always \
  --volume /Users/stephen/gitlab.orig/config:/etc/gitlab \
  --volume /Users/stephen/gitlab.orig/logs:/var/log/gitlab \
  --volume /Users/stephen/gitlab.orig/data:/var/opt/gitlab \
  --shm-size 2g \
  gitlab/gitlab-ce:latest
```

### Step 4 - Update the Hosts File ###

Update the /etc/hosts file on your laptop to point the hostname specified above to your internal IP address. I’ve added a line that looks like this to accommodate;

```
192.168.1.76 gitlab.perciballi.ca
```

### Step 5 - Initial Root Password ###

You can obtain the root password so that you’re able to login to GitLab locally.

First, find the container ID with;

```
docker ps -a
```

Next, get the root password by replacing ea92604a1d7e with your specific container ID.

```
docker exec -it ea92604a1d7e grep 'Password:' /etc/gitlab/initial_root_password
```

### Step 6 - Register a New User ###

Navigate to the hostname specified in step 3 in a browser.

Register new user.

As the root user, navigate to /admin and approve users.

### Step 7 - TLS Certificate ###

Creating a TLS certificate is required so that it is associated with your domain, even if that domain (i.e. gitlab.perciballi.ca) only resolves locally. If you’re looking to use the CI functionality using local runners in the next step you will certainly need to follow these steps.

First, find the container ID with;

```
docker ps -a
```

Next, get the root password by replacing ea92604a1d7e with your specific container ID.

```
docker exec -it ea92604a1d7e /bin/bash
```

Change into the Gitlab configuration directory

```
cd /var/opt/gitlab/nginx/conf
```

Create a file called ```localhost.conf``` and put something similar to the following in it, making sure to change the value of DNS.1 to your domain.

```
[req]
default_bits       = 2048
default_keyfile    = localhost.key
distinguished_name = req_distinguished_name
req_extensions     = req_ext
x509_extensions    = v3_ca

[req_distinguished_name]
countryName                 = Country Name (2 letter code)
countryName_default         = CA
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Ontario
localityName                = Locality Name (eg, city)
localityName_default        = Toronto
organizationName            = Organization Name (eg, company)
organizationName_default    = Perciballi
organizationalUnitName      = organizationalunit
organizationalUnitName_default = Development
commonName_max              = 64

[req_ext]
subjectAltName = @alt_names

[v3_ca]
subjectAltName = @alt_names

[alt_names]
DNS.1   = gitlab.perciballi.ca
```

Create the certificate

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt -config ./localhost.conf
```

Take a look at gitlab-http.conf and identify the ssl_certificate and ssl_certificate_key to move the original certificates generated out of the way. After rebooting Gitlab sets the path back to these default paths so it’s better to move the files and copy the new ones in.

```
mv /etc/gitlab/ssl/gitlab.perciballi.ca.crt /etc/gitlab/ssl/gitlab.perciballi.ca.crt.orig
mv /etc/gitlab/ssl/gitlab.perciballi.ca.key /etc/gitlab/ssl/gitlab.perciballi.ca.key.orig
```

Copy the certificate and key to the system directories. You can put these wherever you want, but make sure to take note of the path for the next step.

```
cp localhost.crt /etc/gitlab/ssl/gitlab.perciballi.ca.crt
cp localhost.key /etc/gitlab/ssl/gitlab.perciballi.ca.key
```

Finally restart GitHub with gitlab-ctl restart or just nginx gitlab-ctl restart nginx

### Step 8 - GitLab Runner ###

Runners connect a compute resource to the SCM to provide a place to run commands. For example if you set an instruction in the CI tool to clone a repo and build a container from that source all of that has to happen somewhere. By deploying a runner on your local machine this is where all of those commands will get executed.

The instructions are now available at ```https://<your instance domain>/admin/runners.``` Beside the 

New Instance Runner button there are elipses indicating a menu where you can see the new runner instructions. Today they look like this. Make sure the system paths are correct. I typically download the runner to $GITLAB_HOME;
Download the binary for your system

```
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
```

Give it permission to execute

```
sudo chmod +x /usr/local/bin/gitlab-runner
```

Create a GitLab Runner user

```
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```

Install and run as a service

```
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

### Troubleshooting ###

One of the most handy commands will be to look at the container logs. In the following command the -f gitlab is the name you gave the container in step 3.

```
sudo docker logs -f gitlab (or the container ID instead of gitlab)
```

Gitlab recommends running many of the commands and the container as root (yuck). I’ve had issues with the container starting with the following in the logs.

```
==> /var/log/gitlab/gitlab-workhorse/current <==
{"correlation_id":"","duration_ms":0,"error":"badgateway: failed to receive response: dial unix /var/opt/gitlab/gitlab-rails/sockets/gitlab.socket: connect: operation not supported","level":"error","method":"GET","msg":"","time":"2024-03-05T20:45:12Z","uri":""}
```

Removing the file with ```rm /var/opt/gitlab/gitlab-rails/sockets/gitlab.socket``` was the only solution as chown did not work.