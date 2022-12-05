# django-kub8

### + Production Deployment Of Django REST API using Kubernetes, Github Actions etc

++ Setup Environment 

- Python 3.10
- Kubernetes
- Docker
- Doctl


### % Kickst*art the project* 

#### -- Create django project env:

    $ python3 -m venv venv
    
#### -- Activate Source Env:

    $ source venv/bin/activate

#### -- Upgrade PIP:
    
    $ pip install pip --upgrade

#### -- Install requirements:
    $ mkdir web & cd web
    $ touch requirments.txt
    """
        django==4.1.3
        gunicorn==20.1.0
        requests==2.28.1
        django-dotenv==1.4.1
        psycopg2-binary==2.9.5
        django-storages==1.13.1
        boto3==1.26.19
    """
    $ pip install -r requirments.txt

#### -- Create Django Project (Inside `web` directory):

    $ django-admin startproject django_kub8 .

#### -- Running the webserver using Docker : 

    - Prod Version (Gunicorn):

    $ /opt/venv/bin/gunicorn --worker-tmp-dir /dev/shm django_kub8.wsgi:application --bind "0.0.0.0":${APP_PORT}
    
    - Local Version (python3):

    $ python3 manage.py runserver

#### -- Connecting to Kubernetes Cluster Provisioned on DigitalOcean :
    
    1- go to https://cloud.digitalocean.com/kubernetes/
    2- Select your cluster `django-k8s`.
    3- Click on Actions -> Download Config
    4- put this config file inside a directory called `.kub`  that you should create under the parent directory. (/django-kub8)
    5- determine the path of your `django-k8s-kubeconfig.yaml` file.
    6- go to your own terminal -> 
        $ export KUBECONFIG=~/Desktop/django-kub8/.kub/django-k8s-kubeconfig.yaml
    
    7- now you have access to your kubernetes cluster nodes : 
    
    $ kubectl get nodes
    
    NAME                     STATUS   ROLES    AGE   VERSION
    django-kub8-pool-mo967   Ready    <none>   14m   v1.24.4
    django-kub8-pool-mo96c   Ready    <none>   14m   v1.24.4

#### -- Create Docker Images Registry (DockerHUB) in DigitalOcean :

    1- go to container registery in your digitalocean workspace and create one.
    2- go to the menu and click on `API`.
    3- create a token named `docker-registry-local` then copy the token created.
    4- go to the terminal and enter this command.
    
    $  docker login registry.digitalocean.com
    $ username : <token>
    $ password : <token>

    DONE !

#### -- Build and Push your Docker Image to your Docker Registry :
    
    $ docker build -t registry.digitalocean.com/private-registery/django-kub8:latest -f Dockerfile .
    
    % registry.digitalocean.com/private-registery/<Docker Image Name> is the link to our Docker registry a docker image hosted always should start 
    with the link of it's Docker registry.

    $ docker push registry.digitalocean.com/private-registery/django-kub8 --all-tags

    --all-tags : option that allows us to push all tags that we have in local.

    + go to `Container Registery` -> `private-registery` -> you will be able to find your docker image named `django-kub8`.

    