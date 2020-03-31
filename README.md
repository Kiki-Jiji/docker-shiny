# Shiny Docker

An hello world shiny app that implements Docker

## Docker
To run in a docker container you need to build and then start the container.

`docker build -t shiny .`

`docker run -p 8000:80 shiny`

The above example exposes port 80 in the container on port 8000 in the host, so you will access the application on 8000, even though internally the container is using port 80.
