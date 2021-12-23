# Introduction

This repository contains a helm chart for deploying 

-  a repository secret
-  some istio configurations (enabling isolation and mtls)

in a kubernetes namespace. It makes sense to split up configurations / environment variables / common configurations concerning infrastructure to a separate helm chart. These things will not change that much like your application service (backend or frontend). You could also add things like connection strings to resources like hubs, vaults or databases (there you should use kubernetes secrets) as part of yaml files. In your backend or frontend applications you can then mount those configurations/secrets or try to call the environment variables in order to use them.

In the following I'm going to explain how to use this chart and how to get / set the information you need. 

---

## Container Repository

There is a `pull-secret-yaml` file which is going to create the secret with our `dockerconfigjson`. You have to set the value in the `values.yaml` file or you're going to fill it from the outside by using your CI/CD Tool like Jenkins, Argo or Artifactory. 

You will find your `dockerconfigjson` in your local directory on Windows: "C:\Users\your_user\.docker"  
The `config.json` file will be filled with your login credentials after you loggedin via the CLI locally. If you cannot see your value you have maybe to check your `credStore` setting. If it is set to "desktop" you will not see any secret, it is stored in Docker Desktop. You have to remove this part and login again via the CLI. You can also take a look here: https://github.com/docker/docker-credential-helpers/issues/95 You should then see the the credential and copy it. 

You can also execute the the following Kubernetes CLI command in an already existing namespace (in an remote Cluster or locally via Docker Desktop for examle) for creating the secret and extracing the base64 encoded credential. 

```
kubectl create secret generic regcred --from-file=.dockerconfigjson=config.json --type=kubernetes.io/dockerconfigjson
```

So you can make sure that you have exactly the same format you will later use.

---

## Istio

Some Kubernetes Cluster allow enabling Istio Injection via own Yaml file. Some Kubernetes Cluster do not allow to do this automatically. Then the Kubernetes CLI has to be used. The command for enabling the injection is: `kubectl label namespace your_namespace istio-injection=enabled --overwrite`

Here you can find a `namespace-isolation.yaml` for enabling the isolation by file. You can use it by setting `enableByYaml` to `true` in the `values.yaml` file. You can also enable the mtls configuration then by setting `enabled` to `true` in the mtls part of the `values.yaml` file. This will then use the `namespace-tmls.yaml` for creating a policy in order to use mtls for service to service encryption in your namespace. 