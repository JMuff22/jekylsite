# Jekyll Site with Openshift

Currently does not work with Ubuntu. But works with `rockylinux:8` !


Test the build locally with `docker build -t test-jekyll .` but will get stuck at the `git clone` stage. This is fine.

Occasionally the build will fail for example:

```
Build Config Instantiate Failed — Build Config test-jekyll
```

This is likely due to hitting a pull limit. To get around this add your own docker hub credentials to the project on rahti. https://docs.csc.fi/cloud/rahti/tutorials/docker_hub_login/


```
oc -n test-jekyll secrets link builder test-jekyll --for=pull
oc -n test-jekyll secrets link deployer test-jekyll --for=pull
oc -n test-jekyll secrets link default test-jekyll --for=pull
```


Run on Rahti/Openshift with 

```
oc new-app https://github.com/JMuff22/jekylsite --name test-jekyll
```

Liveness and readiness probes are used for restarting pods and for checking if new containers function properly before leading traffic to them:

    A liveness probe checks if a container is actually running. A failed liveness check will usually result in the container being killed.
    A readiness probe checks if the container functions properly and can receive requests. A failed readiness probe results in OpenShift halting traffic to the container.

In our case, we choose to probe the same endpoint:


```
oc set probe dc/test-jekyll --liveness --get-url=http://:8000/test-jekyll/index.html
oc set probe dc/test-jekyll --readiness --get-url=http://:8000/test-jekyll/index.htm
```

Then expose the service via `oc expose service test-jekyll`

Check the pod on: https://rahti.csc.fi:8443/console/project/test-jekyll/overview
View the deployment at: http://test-jekyll7-test-jekyll.rahtiapp.fi/

Source: https://www.redpill-linpro.com/sysadvent/2017/12/10/jekyll-openshift.html

Setting up OC CLI: https://docs.csc.fi/cloud/rahti/usage/cli/


## Deploy on Github action / push

Rahti --> Builds --> Builds --> App name --> configuration --> GitHub Webhook URL 

Copy and paste the URL into Github repo webhook. The secret is already attached. 
