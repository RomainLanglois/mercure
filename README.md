# Mercure

[![Build Status](https://travis-ci.org/synhack/mercure.svg?branch=master)](https://travis-ci.org/synhack/mercure)
[![Coverage Status](https://coveralls.io/repos/github/synhack/mercure/badge.svg?branch=master)](https://coveralls.io/github/synhack/mercure?branch=master)
[![Documentation Status](https://readthedocs.org/projects/mercure/badge/?version=latest)](http://doc.mercure.io/en/latest/?badge=latest)
[![Code Health](https://landscape.io/github/synhack/mercure/master/landscape.svg?style=flat)](https://landscape.io/github/synhack/mercure/master)
[![Requirements Status](https://requires.io/github/synhack/mercure/requirements.svg?branch=master)](https://requires.io/github/synhack/mercure/requirements/?branch=master)

Mercure is a tool for security managers who want to teach their colleagues about phishing.


## What Mercure can do:

* Create email templates
* Create target lists
* Create landing pages
* Handle attachments
* Let you keep track in the Campaign dashboard
* Track email reads, landing page visits and attachment execution.
* Harvest credentials
* Schedule campaigns
* Minimize link in email templates


## What Mercure will do:

* Display more graphs (we like graphs!)
* Provide a REST API
* Allow for multi-message campaigns (aka scenarios)
* Check browser plugins
* User training

# Docker Quickstart

## Requirements

* docker
* docker-compose

## Available configuration

| Environment variable name | Status   | Description                              | Value example                      |
| ------------------------- | -------- | ---------------------------------------- | ---------------------------------- |
| SECRET_KEY                | Required | Django secret key                        | Random string                      |
| URL                       | Required | Mercure URL                              | https://mercure.example.com        |
| EMAIL_HOST                | Required | SMTP server                              | mail.example.com                   |
| EMAIL_PORT                | Optional | SMTP port                                | 587                                |
| EMAIL_HOST_USER           | Optional | SMTP user                                | phishing@example.com               |
| EMAIL_HOST_PASSWORD       | Optional | SMTP password                            | P@SSWORD                           |
| REDIS_HOST                | Optional | Redis server                             | 127.0.0.1                          |
| REDIS_PORT                | Optional | Redis port                               | 6379                               |
| DEBUG                     | Optional | Run on debug mode                        | True                               |
| SENTRY_DSN                | Optional | Send debug info to sentry.io             | https://23xxx:38xxx@sentry.io/1234 |
| AXES_LOCK_OUT_AT_FAILURE  | Optional | Ban on forcebrute login                  | True                               |
| AXES_COOLOFF_TIME         | Optional | Ban duration on forcebrute login (in hours) | 0.8333                             |
| DONT_SERVES_STATIC_FILE   | Optional | Don't serve static files with django     | True                               |


## Sample deployment

Create a file, call it "docker-compose.yml" and copy the following code 

```yaml
version: '2'

services:
  redis:
    image: redis
    restart: always
    volumes:
      - /etc/localtime:/etc/localtime:ro
  front:
    image: synhackfr/mercure
    restart: always
    links:
      - redis:redis
    ports:
      - 8000:8000
    volumes:
      - ./data/db:/usr/src/app/data
      - ./data/media:/usr/src/app/media
      - /etc/localtime:/etc/localtime:ro
    environment:
      - SECRET_KEY=<random value>
      - EMAIL_HOST=mail.example.com
      - EMAIL_PORT=587
      - EMAIL_HOST_USER=phishing@example.com
      - EMAIL_HOST_PASSWORD=P@SSWORD
```

To generate the SECRET_KEY variable, you can use this command:

```shell
# generate random SECRET_KEY
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 200 | head -n 1
```

The SECRET_KEY is used as a salt for django password hashing, don't change it after using it with mercure.
After changing the secret key, you can run the container with this command:

```shell
docker-compose up -d
```

Next, you can create a super user to log into web interface:

```bash
# create super user
docker-compose exec front python manage.py createsuperuser
```

# Git Quickstart

## Requirements

* python >= 3.5
* pip

## Deployment

At first remember that Mercure is only compatible with Python 3. When using pip and manage.py ensure that ```pip -V``` and ```python -V``` are Python3 versions. You can use *virtualenv* to define python 3 as the default version for a project without changing system wide version
You need also to install docker and redis database


```
# Start docker.service
sudo systemctl start docker.service

git clone https://github.com/synhack/mercure.git && cd mercure
pip install -r requirements.txt
./manage.py makemigrations
./manage.py migrate
./manage.py collectstatic
./manage.py createsuperuser

# In three different tabs
docker run --name redis -d -p 6379:6379 redis
./manage.py runserver
./manage.py rqworker default
./manage.py rqscheduler
```

# How to use mercure

We can consider mercure is divided into 5 steps :
* Targets
* Email Templates
* Campaigns
* Attachments
* Landing page

Targets, Email Templates and Campaign are the minimum required to run a basic phishing campaign.


1. First, add your targets

   ![Targets](docs/img/mercure_targets.png)

   You need to fill mercure name, the target email.Target first and last name are optional, but can be usefull to the landing page

2. Then, fill the email template.

   ![Landing page](docs/img/mercure_emailtemplate.png)

   You need to fill the mercure name, the subject, the send and the email content.
   To improve the email quality, you have to fill the email content HTML and the text content.
   To get information about opened email, check "Add open email tracker"
   You can be helped with "Variables" category.

   Attachments and landing page are optionnal, we will see it after.

3. Finally, launch the campaign

   ![Campaign](docs/img/mercure_campaign.png)

   You need to fill the mercure name, select the email template and the target group.
   You can select the SMTP credentials, SSL using or URL minimazing

   Note : When filling the smtp form, don't forget to specify the smtp hostname and the port.
   
   For example :  smtp.gmail.com:587

4. Optional, add landing page

   ![Landing page](docs/img/mercure_landingpage.png)

   You need to fill the mercure name, the domain to use
   You can use "Import from URL" to copy an existing website.

   You have to fill the page content with text and HTML content by clicking to "Source"

5. Optional, add Attachment

   ![Attachments](docs/img/mercure_attachment.png)

   You need to fill the mercure name, the file name which appears in the email and the file
   You also have to check if the the file is buildable or not, if you need to compute a file for example.

   To execute the build , you need to create a zip archive which contain a build script (named 'generator.sh' and a buildable file



# Developers

## To participate to the project :

1. Fork the project

2. Create new branch

3. Make comments and clean commits to the repository

4. Run unnittests
  ```
  docker run --name redis -d -p 6379:6379 redis
  python manage.py test --exclude-tag selenium
  ```

5. Perform a pull request


# Responsible Disclosure of Security Vulnerabilities

We want to keep Mercure safe for everyone. If you've discovered a security vulnerability in Mercure, we appreciate your help in disclosing it to us in a responsible manner.

Send an email to 'security@synhack.fr'. If you want, you can use with [PGP Key](https://pgp.mit.edu/pks/lookup?op=vindex&search=security@synhack.fr)


## Vulnerability summary

* Name of the vulnerability
* Attack Vector (AV)
* Attack Complexity (AC)
* Privileges Required (PR)
* User Interaction (UI)
* Scope (S)
* Confidentiality (C)
* Integrity (I)
* Availability (A)


## Reporter informations
* Your Name
* Your Mail
* Your PGP public key


## Technical details
* More technical details.
