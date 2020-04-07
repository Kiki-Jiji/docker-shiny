# Shiny on GCP App Engine

This is a walkthrough of getting a shiny app hosted on Google Cloud Platform

In theory if you have a Docker container with a Shiny app within, it should be able to run on Google App Engine, where advantages include management of run time and launching on demand, rather than having to run a Shiny server all the time.

## GCP App engine
App Engine doesn't offer a default R environment so you will need to define your own custom runtime. You can see the GCP App Engine [Building Custom Runtimes](https://cloud.google.com/appengine/docs/flexible/custom-runtimes/build?hl=en_US#listening_to_port_8080) documentation for more details.

Importantly you will need to:
* Add a `Dockerfile` that internally configures the runtime environment. Comprehensive documentation on creating Dockerfiles is available on the [Docker website](https://docs.docker.com/engine/reference/builder/).
* Provide an `app.yaml` file that describes your application's runtime configuration to App Engine.
* Ensure that your code follows some basic rules. The main element to consider is that the App Engine front end will route incoming requests to the appropriate module on port 8080. You must be sure that your application code is listening on 8080. This will need to be specified in the `Dockerfile` as `EXPOSE 8080`.

## Requirements
* GCP Account
* GCP SDK
  - If you don't have this already you can [download here](https://cloud.google.com/sdk/docs/quickstarts)
  - Alternatively you can use the GCP cloud terminal
* This repository
  - To run a different ShinyApp just replace the folder App with your ShinyApp (but still called App). For this walkthrough the additional files on-top of shiny are `Dockerfile`, `app.yaml`, `shiny-server.conf` & `shiny-server.sh`.
  * **optional** To run locally you will need to [setup your local development enviroment](https://cloud.google.com/appengine/docs/flexible/custom-runtimes/download)

## Deployment Steps
### 1. Docker: With this repository you should be able to clone, build and run locally.

`git clone github::`

`docker build -t shiny .`

`docker run -p 8000:8080 shiny`

You should now be able to open in a browser (e.g google chrome) `localhost:8000` and see the shiny App.

### 2. Setup a GCP project
You will need:
* A GCP project, see [Creating and Managing Projects](https://cloud.google.com/resource-manager/docs/creating-managing-projects) if you don't have one.

### Deploy

In the terminal run:

```{bash}

  gcloud app deploy

```

By default, the command deploys the app.yaml configuration file from the current directory. If you're running the command from a directory that does not contain you app's app.yaml then:

```{bash}

  gcloud app deploy [CONFIGURATION_FILES]

```
Replace [CONFIGURATION_FILES] with the path to one or more configuration files. Use a single white space to separate pathnames.

Then follow the instruction in the terminal, choosing account, location, project.
> Only deploy to projects with app engine enabled for regions supported by flexible environment you may need to create a new Google project for this as project region can not be changed once initialised.

This will then build... slowly...

![](https://media.makeameme.org/created/The-slow-service.jpg)

But after a few minutes the app will be deployed! Run:
```bash
  gcloud app browse
```
You should get the message (if the project is called shiny-test),
`Opening [https://shiny-test-273509.appspot.com] in a new tab in your default browser.`

Once deployed the app within is shown at https://your-project.appspot.com/shiny/ with the Shiny help page at https://your-project.appspot.com/


### Notes
To include more dependencies this can be stated in the `Dockerfile` on the line

`RUN R -e "install.packages(c('shiny'), repos='http://cran.rstudio.com/')"`

To add the package `shinydashboard` add to the `install.packages()` command.  

`RUN R -e "install.packages(c('shiny', 'shinydashboard'), repos='http://cran.rstudio.com/')"`

Breaks due to Web Sockets not being proxied https://support.rstudio.com/hc/en-us/articles/213733868-Running-Shiny-Server-with-a-Proxy

https://github.com/rstudio/shiny-server/issues/155#issuecomment-231883143

The proxy used currently is this one:
https://github.com/GoogleCloudPlatform/appengine-sidecars-docker/tree/master/nginx_proxy

Nginx required:

```
http {

  map $http_upgrade $connection_upgrade {
      default upgrade;
      ''      close;
    }

  server {
    listen 80;



    location / {
      proxy_pass http://localhost:3838;
      proxy_redirect http://localhost:3838/ $scheme://$host/;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_read_timeout 20d;
    }
  }
}
``
