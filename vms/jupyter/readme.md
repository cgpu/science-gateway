- [Creating  a JupyterHub on Jetstream with the Zero to JupyterHub Project](#h-D73CBC56)
  - [Kubernetes Cluster](#h-65F9358E)
    - [jupyterhub.sh](#h-B56E19AB)
    - [Create Cluster](#h-2FF65549)
  - [unidata/unidatahub Docker Container](#h-CD007D2A)
  - [Configure and Deploy the JupyterHub](#h-E5CA5D99)
    - [Letsencrypt versus Certificate from a Certificate Authority](#h-294A4A20)
    - [OAuth Authentication](#h-8A3C5434)
    - [unidata/unidatahub](#h-214D1D4C)
  - [Navigate to JupyterHub](#h-209E2FBC)
  - [Tearing Down JupyterHub](#h-1E027567)
    - [Total Destructive Tear Down](#h-A69ADD92)
    - [Tear Down While Preserving User Volumes](#h-5F2AA05F)
  - [Troubleshooting](#h-0E48EFE9)
    - [Unresponsive JupyterHub](#h-FF4348F8)



<a id="h-D73CBC56"></a>

# Creating  a JupyterHub on Jetstream with the Zero to JupyterHub Project


<a id="h-65F9358E"></a>

## Kubernetes Cluster


<a id="h-B56E19AB"></a>

### jupyterhub.sh

`jupyterhub.sh` and the related `z2j.sh` are convenience scripts similar to `openstack.sh` to give you access to a pre-configured environment that will allow you to build and/or run a Zero to JupyterHub cluster. It also relies on the [same Docker container](../../openstack/readme.md#h-4A9632CC) as the `openstack.sh` script. `jupyterhub.sh` takes one argument with the `-n` option, the name of the Zero to JupyterHub cluster. Invoke it from the `science-gateway/openstack` directory. `jupyterhub.sh` and the related `z2j.sh` ensure the information for this Zero to JupyterHub cluster is persisted outside the container via Docker file mounts &#x2013; otherwise all the information about this cluster would be confined in memory inside the Docker container. The vital information will be persisted in a local `jhub` directory.


<a id="h-2FF65549"></a>

### Create Cluster

[Create a Kubernetes cluster](../../openstack/readme.md) with the desired number of nodes and VM sizes. Lock down the master node of the cluster per Unidata security procedures. Work with sys admin staff to obtain a DNS name (e.g., jupyterhub.unidata.ucar.edu), and a certificate from a certificate authority for the master node.


<a id="h-CD007D2A"></a>

## unidata/unidatahub Docker Container

Build the Docker container in this directory and push it to dockerhub.

```sh
docker build -t unidata/unidatahub:`openssl rand -hex 6` . > /tmp/docker.out 2>&1 &
docker push unidata/unidatahub:<container id>
```


<a id="h-E5CA5D99"></a>

## Configure and Deploy the JupyterHub

From the client host where you created the Kubernetes cluster, follow [Andrea Zonca's instructions](https://zonca.github.io/2018/09/kubernetes-jetstream-kubespray-jupyterhub.html).

After you have created the `secrets.yaml` as instructed, customize it with the choices below


<a id="h-294A4A20"></a>

### Letsencrypt versus Certificate from a Certificate Authority

1.  Letsencrypt

    Follow [Andrea's instructions](https://zonca.github.io/2018/09/kubernetes-jetstream-kubespray-jupyterhub.html) on setting up letsencrypt along with this yaml snippet below. Replace the hostname where appropriate.

    ```yaml
    ingress:
      enabled: true
      annotations:
        kubernetes.io/tls-acme: "true"
      hosts:
        - <jupyterhub-host>
      tls:
          - hosts:
             - <jupyterhub-host>
            secretName: certmanager-tls-jupyterhub
    ```

2.  Certificate from CA

    Work with sys admin staff to obtain a certificate from a CA.

    Follow [Andrea's instructions](https://zonca.github.io/2018/09/kubernetes-jetstream-kubespray-jupyterhub.html) on setting up HTTPS with custom certificates. Note that when adding the key with

    ```shell
    kubectl create secret tls <cert-secret> --key ssl.key --cert ssl.crt -n jhub
    ```

    supply the regular and not full chain certificate.

    Here is a snippet of what the ingress configuration will look like in the `secrets.yaml`.

    ```yaml
    ingress:
      enabled: true
      annotations:
        kubernetes.io/tls-acme: "true"
      hosts:
        - <jupyterhub-host>
      tls:
          - hosts:
             - <jupyterhub-host>
            secretName: <secret_name>
    ```


<a id="h-8A3C5434"></a>

### OAuth Authentication

1.  Globus

    [Globus OAuth capability](https://developers.globus.org/) is available for user authentication. The instructions [here](https://github.com/jupyterhub/oauthenticator#globus-setup) are relatively straightforward.

    ```yaml
    auth:
      type: globus
      globus:
        clientId: "xxx"
        clientSecret: "xxx"
        callbackUrl: "https://<jupyterhub-host>:443/oauth_callback"
        identityProvider: "xsede.org"
      admin:
        users:
          - adminuser1
    ```

2.  GitHub

    Setup an OAuth app on GitHub

    ```yaml
    auth:
      type: github
      github:
        clientId: "xxx"
        clientSecret: "xxx"
        callbackUrl: "https://<jupyterhub-host>:443/oauth_callback"
      admin:
        users:
          - adminuser1
    ```


<a id="h-214D1D4C"></a>

### unidata/unidatahub

Add the Unidata JupyterHub configuration (`unidata/unidatahub`) and related items (e.g., pulling of Unidata Python projects). Customize the desired CPU / RAM usage. [This spreadsheet](https://docs.google.com/spreadsheets/d/15qngBz4L5gwv_JX9HlHsD4iT25Odam09qG3JzNNbdl8/edit?usp=sharing) will help you determine the size of the cluster based on number of users, desired cpu/user, desired RAM/user. Duplicate it and adjust it for your purposes.

```yaml
  type: github
  github:
    clientId: "xxx"
    clientSecret: "xxx"
    callbackUrl: "https://jupyterhub.unidata.ucar.edu:443/oauth_callback"
  admin:
    users:
      - admin
  whitelist:
    users:
      - user

singleuser:
  storage:
    capacity: 10Gi
  startTimeout: 600
  memory:
    guarantee: 8G
    limit: 8G
  cpu:
    guarantee: 3
    limit: 4
  defaultUrl: "/lab"
  image:
    name: unidata/unidatahub
    tag: b2e5978575d8
  lifecycleHooks:
    postStart:
      exec:
          command:
            - "sh"
            - "-c"
            - >
              gitpuller https://github.com/Unidata/python-training master python-training;
              cp /README_FIRST.ipynb /home/jovyan;
              cp /.condarc /home/jovyan

hub:
  extraConfig: |-
```


<a id="h-209E2FBC"></a>

## Navigate to JupyterHub

In a web browser, navigate to your newly minted JupyterHub and see if it is as you expect.


<a id="h-1E027567"></a>

## Tearing Down JupyterHub


<a id="h-A69ADD92"></a>

### Total Destructive Tear Down

Tearing down the JupyterHub including user OpenStack volumes is possible. From the Helm and Kubernetes client:

```sh
helm delete jhub --purge
kubectl delete namespace jhub
```

To further tear down the Kubernetes cluster see [Tearing Down the Cluster](../../openstack/readme.md).


<a id="h-5F2AA05F"></a>

### Tear Down While Preserving User Volumes

A gentler tear down that preserves the user volumes is described in [Andrea's documentation](https://zonca.github.io/2018/09/kubernetes-jetstream-kubespray-jupyterhub.html). See the section on "persistence of user data".


<a id="h-0E48EFE9"></a>

## Troubleshooting


<a id="h-FF4348F8"></a>

### Unresponsive JupyterHub

1.  Preliminary Work

    If a JupyterHub becomes unresponsive (e.g., 504 Gateway Time-out), login in to the Kubernetes client and do preliminary backup work in case things go badly. First:

    ```shell
    kubectl get pvc -n jhub -o yaml > pvc.yaml.ro
    kubectl get pv -n jhub -o yaml > pv.yaml.ro
    chmod 400 pvc.yaml.ro pv.yaml.ro
    ```

    Make `pvc.yaml.ro` `pv.yaml.ro` read only since these files could become precious in case you have to do data recovery for users. More on this subject below.

2.  Delete jhub Pods

    Next, start investigating by issuing:

    ```shell
    kubectl get pods -n jhub
    ```

    this command will yield something like

    ```shell
    NAME                      READY   STATUS    RESTARTS   AGE
    hub-5bdccd4784-lzw87      1/1     Running   0          17h
    jupyter-joe               1/1     Running   0          4h51m
    proxy-7b986cdb75-mhl86    1/1     Running   0          29d
    ```

    Now start deleting the `jhub` pods starting with the user pods (e.g., `jupyter-joe`).

    ```
    kubectl delete pod <pod name> -n jhub
    ```

    Check to see if the JupyterHub is reachable. If it is not, keep deleting pods checking for reachability after each pod deletion.

3.  Delete jhub, But Do Not Purge Namespace

    If the JupyterHub is still not reachable, you can try deleting and recreating the JupyterHub but **do not** delete the namespace as you will wipe out user data.

    ```shell
    helm delete jhub --purge
    # But DO NOT issue this command
    # kubectl delete namespace jhub
    ```

    Then try reinstalling with

    ```
    bash install_jhub.sh
    ```

    Now, try recover user volumes as [described at the end of the section here](https://zonca.dev/2018/09/kubernetes-jetstream-kubespray-jupyterhub.html#delete-and-recreate-openstack-instances) with the `pvc.yaml.ro` `pv.yaml.ro` saved earlier (make writable copies of those `ro` files). If that still does not work, you can try destroying the entire cluster and recreating it as described in that same link.
