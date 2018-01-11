# quandl-jupyter-risk-modeling

This quickstart consists of two microservices:

1. quandl: Fetches and stores data from [Quandl](https://www.quandl.com/) and stores them in Hasura. 

2. jupyter: Runs [Jupyter](https://jupyter.org/) iPython notebook with scipy (and other data analysis libraries) installed for building, analysing and visualising models interactively.

Follow along below to get the setup working on your cluster and also to understand how this quickstart works.

## Prerequisites

* Ensure that you have the [hasura cli](https://docs.hasura.io/0.15/manual/install-hasura-cli.html) tool installed on your system.

```sh
$ hasura version
```

Once you have installed the hasura cli tool, login to your Hasura account

```sh
$ # Login if you haven't already
$ hasura login
```


* You should also have [git](https://git-scm.com) installed.

```sh
$ git --version
```

## Getting started

```sh
$ # Run the quickstart command to get the project
$ hasura quickstart hasura/quandl-jupyter-risk-modeling

$ # Navigate into the Project
$ cd quandl-jupyter-risk-modeling
```

## Quandl

Before you begin, head over to [Quandl](https://www.quandl.com/) and select the dataset you would like to use. In this case, we are going with the `Wiki EOD Stock Prices` dataset. Keep in mind the `Vendor Code` (In this case it is, `WIKI`) and `Datatable Code` (`PRICES` in this case) for the dataset.

![Quandl 1](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/quandl1.png "Quandl 1")

![Quandl 3](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/quandl3.png "Quandl 3")


To fetch the data you need to have an `API Key` which you can get by getting an account with Quandl.

![Quandl 2](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/quandl2.png "Quandl 2")

Keep a note of your `API Key`.

### Adding the Quandl `API Key` to Hasura secrets

Sensitive data like API keys, tokens etc should be stored in Hasura secrets and then accessed as an environment variable in your app. Do the following to add your Quandl API Key to Hasura secrets.

```sh
$ # Paste the following into your terminal
$ # Replace <API-KEY> with the API Key you got from Quandl
$ hasura secret update quandl.api.key <API-KEY>
```

This value is injected as an environment variable (QUANDL_API_KEY) to the quandl service like so:

```yaml
env:
- name: QUANDL_API_KEY
  valueFrom:
  secretKeyRef:
    key: quandl.api.key
    name: hasura-secrets
```

Check your `k8s.yaml` file inside `microservices/quandl/app` to check out the whole file.

Next, let's deploy the app onto your cluster.

## Deploy app

`Note: Deploy will not work if you have not followed the previous steps correctly`

```sh
$ # Ensure that you are in the quandl-jupyter-risk-modeling directory
$ # Git add, commit & push to deploy to your cluster
$ git add .
$ git commit -m 'First commit'
$ git push hasura master
```

Once the above commands complete successfully, your cluster will have two services `jupyter` and `quandl` running. To get their URLs

```sh
$ # Run this in the quandl-jupyter-risk-modeling directory
$ hasura microservice list
```

```sh
• Getting microservices...
• Custom microservices:
NAME       STATUS    INTERNAL-URL       EXTERNAL-URL
jupyter    Running   jupyter.default    http://jupyter.boomerang68.hasura-app.io
quandl     Running   quandl.default     http://quandl.boomerang68.hasura-app.io

• Hasura microservices:
NAME            STATUS    INTERNAL-URL           EXTERNAL-URL
auth            Running   auth.hasura            http://auth.boomerang68.hasura-app.io
data            Running   data.hasura            http://data.boomerang68.hasura-app.io
filestore       Running   filestore.hasura       http://filestore.boomerang68.hasura-app.io
gateway         Running   gateway.hasura
le-agent        Running   le-agent.hasura
notify          Running   notify.hasura          http://notify.boomerang68.hasura-app.io
platform-sync   Running   platform-sync.hasura
postgres        Running   postgres.hasura
session-redis   Running   session-redis.hasura
sshd            Running   sshd.hasura
```

You can access the services at the `EXTERNAL-URL` for the respective service.

## Exploring the data

Currently our database has not gotten any data from quandl. You can head over to your `api console` to check this out. It will have one table called `quandl_checkpoint` which stores the current offset at which the data in Hasura is stored.

```sh
$ # Run this in the quandl-jupyter-risk-modeling directory
$ hasura api-console
```

![Console Data 1](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/console_data_initial.png "Console Data 1")

Let's use our `quandl` service to insert some data. To do this:

```
POST https://quandl.<CLUSTER-NAME>.hasura-app.io/add_data // remember to replace <CLUSTER-NAME> with your own cluster name (In this case, http://quandl.boomerang68.hasura-app.io/add_data)

{
    "vendor_code": "WIKI",
    "datatable_code": "PRICES"
}
```

You can use a HTTP client of your choosing to make this request. Alternatively, you can also use the `API Explorer` provided by the Hasura `api console` to do this.

![Console Data 2](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/console_api_explorer_quandl_api.png "Console Data 2")

There is also a notebook called `fetch` in the jupyter service which makes the API call to fetch and insert data. Read more in jupyter section below.

Once you have successfully made the above API call. Head back to your `api console` and you will see a new table called `wiki_prices` with about 10000 rows of data in it.

![Console Data 3](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/console_data_wiki_prices.png "Console Data 3")

## Analysing the data using Jupyter notebook

### Open the jupyter service

Head over to the EXTERNAL-URL of your `jupyter` service.

![Jupyter 1](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/jupyter_login.png "Jupyter 1")

### Authentication

```sh
$ # Run this in the quandl-jupyter-risk-modeling directory
$ hasura ms logs jupyter
```

Copy the authentication token from the logs and enter it in the jupyter UI above. 

```sh
Executing the command: jupyter notebook
[I 07:14:46.914 NotebookApp] Writing notebook server cookie secret to /home/jovyan/.local/share/jupyter/runtime/notebook_cookie_secret
[W 07:14:47.379 NotebookApp] WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
[I 07:14:47.428 NotebookApp] JupyterLab alpha preview extension loaded from /opt/conda/lib/python3.6/site-packages/jupyterlab
[I 07:14:47.428 NotebookApp] JupyterLab application directory is /opt/conda/share/jupyter/lab
[I 07:14:47.434 NotebookApp] Serving notebooks from local directory: /home/jovyan
[I 07:14:47.434 NotebookApp] 0 active kernels
[I 07:14:47.434 NotebookApp] The Jupyter Notebook is running at:
[I 07:14:47.434 NotebookApp] http://[all ip addresses on your system]:8888/?token=cd596a9b5e90a83283e4c9d6b792b4a58cac38e06153fd12
[I 07:14:47.434 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 07:14:47.435 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=cd596a9b5e90a83283e4c9d6b792b4a58cac38e06153fd12

```


### Risk Modeling

After authenticating, go inside the work folder. You will see two files: `fetch.ipynb` and `risk.ipynb`

![Jupyter 2](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/jupyter_workspace.png "Jupyter 2")


Open `risk.ipynb`

Each cell contains a description of its intent. In short, we load a ticker data and apply the GARCH(1,1) model on it.  

![Jupyter 3](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/jupyter_risk_1.png "Jupyter 3")

![Jupyter 4](https://raw.githubusercontent.com/hasura/quandl-jupyter-risk-modeling/master/assets/jupyter_risk_2.png "Jupyter 4")

And that's it! 





