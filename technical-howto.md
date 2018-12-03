# Intro

This is meant to serve as a how to for setting up a JupyterHub installation through Google Cloud.

Primarily, you should follow the steps in [Zero to JupyterHub](http://zero-to-jupyterhub.readthedocs.io/en/latest/index.html). I'll cover deviations here.

# Step Zero: Setting up Kubernetes on Google Cloud

Follow the instructions [here](http://zero-to-jupyterhub.readthedocs.io/en/latest/google/step-zero-gcp.html).

Follow steps 1-4 as written, then continue to the following section before running step 5.

## Creating the cluster

When you create your Kubernetes cluster (currently step 5) I recommend:

```
gcloud container clusters create <metis-hub-cluster> \
    --num-nodes=1 \
    --machine-type=n1-standard-4 \
    --zone=us-central1-b
```

You should size your `machine-type` to be the smallest size that could reliably run the skeleton of the hub (so the database, and proxy pods). I've found that n1-standard-1 is definitely under-sized and n1-standard-2 probably is as well. Because there's overhead to every new node, you don't want it to require more than 1 node to run just the basics.

In my experience, the zone location doesn't appreciably affect performance, so my recommendation is to pick the cheapest nearby zone, typically `us-central*`.

More info on Google cloud [zones](https://cloud.google.com/compute/docs/regions-zones/#available) and [machine-types](https://cloud.google.com/compute/docs/machine-types). I think it's likely that we could optimize the machine type to better balance CPU and RAM (RAM is the expensive one!) with testing.

Continue with the remaining steps in the Zero to Jupyter-Hub howto.

# Step One: Setting up JupyterHub

Follow the steps in [Setting up JupyterHub](http://zero-to-jupyterhub.readthedocs.io/en/latest/getting-started.html). However, use the config file included [here](config.yaml) as a starting point. I'll cover the items in this file momentarily.

When you get to step 4 (where you run `kubectl --namespace=<YOUR-NAMESPACE> get svc`) note the IP address here. You will most likely wish to use this to set up an A-Record. Mine uses Google's domain service and maps my hub to `https://hub.soph.info`. Here's what my A-Record looks like:

| Name | Type | TTL | DATA     |
|------|------|-----|----------|
| hub | A | 1h | 35.196.121.84 |

Execute any remaining steps.

# Step Two: Customizations

As stated in the Zero to JupyterHub howto, you can update the hub any time you edit the `config.yaml` file with:

```
helm upgrade <YOUR_RELEASE_NAME> jupyterhub/jupyterhub \
    --version=v0.6 \
    -f config.yaml
```

If you wish to update the JupyterHub version, then you'll need to modify this command.

## User Environment: Docker

I highly recommend you use Docker to manage the user environment—the files and installed software that users start with.

My recommended process for this is to:
- Create your own dockerfile. Likely, this should pull from one of the [official Jupyter docker stacks](https://github.com/jupyter/docker-stacks). I've included the one I use here for reference.
- Using DockerHub, set up an automated build that pulls from a github repository. See [mine](https://hub.docker.com/r/sophsea/soph-hub-user/) for reference.
