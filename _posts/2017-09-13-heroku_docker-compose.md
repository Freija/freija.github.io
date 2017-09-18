---
layout: post
title: "Heroku Flask-API Docker with MongoDB"
date: 2017-09-13
tags:
 - docker
 - heroku
 - flask
 - flask api
---
As part of a data-science workshop in August 2016 organized at UC Berkeley, my team and I wrote a recommendation engine that is accessible through a Flask API. The app runs inside a Docker container and was initially deployed on an AWS EC2 instance. This post discusses deploying the code on Heroku using Docker.

## Introduction

The code for the recommendation API and website can be found here: [https://github.com/Freija/recontent](https://github.com/Freija/recontent). The app is now deployed on heroku: [https://recontent.herokuapp.com/](https://recontent.herokuapp.com/).

This is the associated docker-compose file of the app:

```
      version: '2'
      services:
        app:
          build: .
          depends_on:
            - db
          image: "freija/recontent"
          volumes:
            - "./data:/data:ro"
        db:
          image: mongo:3.0.2
          volumes:
            - "./data/db:/data/db"
          command: ["mongod", "--smallfiles"]
```

We have two containers; one to run the Flask API and website (called ```app```) and one to run an instance of MongoDB (called ```db```). MongoDB is used to store information about API requests (response speed, etc) and redirected clicks (delay until link was clicked, etc). The containers have access to a data folder on the host system. This is where we have the database files as well as the model-files of the recommendation algorithms (~500MB). Having the shared folder like that has some advantages:

 * It is easy to access the database from the host system.

 * The model-files can be stored there to avoid having to copy the files into the container.


Now, how to deploy this app on Heroku? It turns out that Heroku supports Docker, see details here: [https://devcenter.heroku.com/articles/container-registry-and-runtime](https://devcenter.heroku.com/articles/container-registry-and-runtime). That reference page lists a few limitations. Most important for this application is that the file system is ephemeral, which means (quoting Heroku documentation): ```no files that are written are visible to processes in any other dyno and any files written will be discarded the moment the dyno is stopped or restarted```. The consequence of this is that volume mounting is not supported. We will need to find a different solution for the shared data folder. In addition, Heroku does not support docker-compose, but it does allow to deploy multiple Docker containers using a ```--recursive``` argument.

After poking around a bit, here is the solution I came up with (other solutions are possible of course) in order to deploy the recommendation app to Heroku:

 1. Copy over the data into the container directly instead of using a shared volume.

 2. Use a database-as-service provider for the MongoDB instance. [mLab](https://mlab.com/) offers a free 0.5GB ['sandbox'](https://mlab.com/plans/pricing/#plan-type=sandbox), which is sufficient for this demo.

That way, I only have one container which connects to the mLab MongoDB instance and has the model-files as part of the build image.

## Details
Note that these steps assume that Docker is already installed.
### Setting up Heroku
[Heroku](https://www.heroku.com/) is a platform-as-service (PaaS), which can be used to deploy web applications. Perfect.

First step is to get a free Heroku account and set up the Heroku Command Line Interface (CLI). The [Python tutorial](https://devcenter.heroku.com/articles/getting-started-with-python) runs through how to do that and also through a small example of how to deploy a Django app. For Ubuntu (my system), the steps to install the Heroku CLI are (quote from tutorial):

```
      $ sudo add-apt-repository "deb https://cli-assets.heroku.com/branches/stable/apt ./"
      $ curl -L https://cli-assets.heroku.com/apt/release.key | sudo apt-key add -
      $ sudo apt-get update
      $ sudo apt-get install heroku
```

Time to go to the directory where the app lives and to log into heroku from the command line as well as the Heroku Container Registry (since we'll be deploying a Docker container):

```
      $ heroku login
      $ sudo heroku:container login
```

On my system, running Docker is set up to require root privileges, hence the ```sudo```. We are now set up to push Docker containers to Heroku. If your code is not set up as a git repository yet, you will need to add a ```git init``` to these steps.

Basically, what will happen when we push a Docker container to Heroku is that the Docker image will be build locally and then pushed to Heroku in layers. This is important for the order in which we do things in the Dockerfile. For example, we will need to copy the model-files to the container. This is a large data-set and therefore should be copied early on in the Dockerfile to avoid copying the data-layer every time we make a change to the app code.


### Tweaking the Dockerfile
The ```app``` container had the following Dockerfile before the migration to Heroku:

```
      FROM python:3.5
      EXPOSE 5010
      COPY requirements.txt /
      RUN pip install -r /requirements.txt
      COPY app/ ./app/
      WORKDIR /data
      USER nobody
      ENTRYPOINT ["python", "../app/app.py"]
```
And the requirements:

```
      flask
      flask-restful
      requests
      justext
      gensim
      pymongo
```

First change is that we will use gunicorn (WSGI HTTP server) to serve the app instead of the Flask built-in web server, which is intended for development. This means adding ```gunicorn``` to the requirements. A Dockerfile ```CMD``` line is required for Heroku, so we will be changing the ```ENTRYPOINT``` line to: ``` CMD gunicorn --bind 0.0.0.0:$PORT wsgi```, where the port-binding comes from the Heroku instructions: ```EXPOSE - While EXPOSE can be used for local testing, it is not supported in Heroku’s container runtime. Instead your web process/code should get the $PORT environment variable.``` The preferred format for the ```CMD``` line is the 'exec' form: ```CMD ["executable","param1","param2"]```. At this time, I have not figured out how to correctly pass the ```$PORT``` variable in that format yet.

Next, as mentioned before, we will copy over the content of the data directory and get rid of the ```EXPOSE``` Dockerfile line. The new Dockerfile looks like this:

```
      FROM python:3.5
      COPY data/ /opt/app/
      COPY requirements.txt /
      RUN pip install -r /requirements.txt
      COPY app/ /opt/app/
      WORKDIR /opt/app
      USER nobody
      CMD gunicorn --bind 0.0.0.0:$PORT wsgi
```

The last thing is to add the ```wsgi.py``` module to the app directory containing the following line:
{% highlight python %}
from app import app as application
{% endhighlight %}

### Setting up MongoDB instance with mLab
Instead of running a MongoDB instance in a second Docker container, a service as mLab can be used. Heroku has mLab as an 'add-on', but I found that, even though the ```sandbox``` tier is free, Heroku requires credit card information to add the mLab MongoDB 'add-on'. No problem, we can spin up a MongoDB instance through the mLab website itself and then just connect to it from within our app. Here are the steps:

 1. Get a [free mLab account](https://mlab.com/welcome/) and start a ```sandbox``` MongoDB instance. The wizard is extremely easy to follow.

 2. From your dashboard on the mLab website, add a user to the database, which has write-rights. For this, click on the database and then go to the ```users``` tab. If you try to connect to the database through the command-line, quick note: make sure the versions of the CL tool and the database are compatible.

 3. Grab the URI from the mLab dashboard for the new database, it will be something like: ```mongodb://<dbuser>:<dbpassword>@ds1424514.mlab.com:41235/recontent``` (fake numbers here).

 4. Make sure the pymongo version is compatible with the database version.

### Setting and using Heroku config variable
Now that we have a MongoDB URI, we need a way to pass on this information to the container. For this, it is easy to use a Heroku config variable. To set the config var use the ```heroku config:set``` command, for example:

```
  $ heroku config:set MONGODB=mongodb://<dbuser>:<dbpassword>@ds1424514.mlab.com:41235/recontent
```

Then run ``` heroku config``` to see the current config variables. The config variable is then accessible as an environment variable inside the container and can be accessed from inside the Python code using the ```os``` module:

{% highlight python %}
  from pymongo import MongoClient
  import os
  database = os.environ.get('MONGODB')
  client = MongoClient(database)
  db = client.get_default_database()
{% endhighlight %}

### Create and run the Heroku app
After the changes have been made, the app is deployed to Heroku:

 1. Make sure we are still logged in:
    ```
      $ heroku login
      $ sudo heroku:container login
    ```
 2. Create the heroku app, 'recontent' is the name in my case:
    ```
      $ heroku apps:create recontent
    ```
 3. The config var MONGODB should be set, check it with:
    ```
      $ heroku config
    ```
 4. Push the container:
    ```
      $ sudo heroku container:push web
    ```
 5. Check the logs:
    ```
      $ heroku logs
    ```
 6. Open the app:
    ```
      $ heroku open
    ```
