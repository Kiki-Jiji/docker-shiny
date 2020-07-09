# Deploy App on cloud run

* build docker container locally, test app works
* Upload docker image to Container Registry
* deploy image from Container Registry on Cloud Run

# Steps

Assume you have an already working docker image. The following steps use the [GCP SDK](https://cloud.google.com/sdk) 

##  Deploy on CLoud Run

Run the following command to build your container and publish on Container Registry.

```bash
gcloud builds submit --tag gcr.io/PROJECT_ID/service_name

gcloud builds submit --tag gcr.io/shiny-test-273509/test-cars --timeout=3600
```
The first line is generic, the second line is an example. The default cloud build time is 10 minutes. When I tested it it took 45 minutes. `--timeout` 
allows you to set a custom time such as an hour (e.g 3600)

You can see what containers are available using
```
gcloud container images list
```


```bash
gcloud run deploy graphviz-web --image gcr.io/PROJECT_ID/graphviz

gcloud run deploy --image gcr.io/shiny-test-273509/test-cars --project shiny-test-273509
```


## 

Useful hint:

```bash
gcloud alpha interactive
``` 
