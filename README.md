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

### -- Provision a PostgresDB Cluster and link it to out Kubernetes Cluster :

    1- go to databases in digitalocean.com.
    2- create postgres database cluster.
    3- click on get started -> choose `django-k8s` kuberenetes cluster to connect to you database cluster.
    4- by then you can see credentials that will allows you to connect to the database.

    - you can use this command below to migrate an existing database to the one you just created.
    $ PGPASSWORD=yAnhtNzqbq3YLc88GS pg_restore -U doadmin -h django-kub8-db-postgresql-do-user-13012526-0.b.db.ondigitalocean.com \
      -p 25060 -d defaultdb <local-pg-dump-path>

#### -- Generate a new secret :

    $ python -c "import secrets; print(secrets.token_urlsafe(32))"
    2WnhYse2eKvT5NrlEDVQN1KCdJQk1hHSscbQMXO6YLQ

#### -- Create Kub8 Secret based on env.prod file:
    
    $ kubectl create secret generic django-kub8-prod-env --from-env-file=web/.env.prod
    $ kubectl get secret
    django-kub8-prod-env   Opaque      

    - Check data stored in this secret 
    $ kubectl get secret django-kub8-prod-env -o YAML

#### -- How to use secret in django-kub8 backend app deployment :

    1- after writing deployment for django-kub8 app to mention that this app gonna use 
       a secret (that we created before look above), where we have env variables (web/.env.prod) for production environment

    you have to mention your Docker registry which is `private-registery`
    $ kubectl get serviceaccount -o YAML
    
    apiVersion: v1
    items:
    - apiVersion: v1
      imagePullSecrets:
      - name: private-registery
    
    in the deployment file for django app
    ```
    imagePullSecrets:
      - name: private-registery
    ```

    2- by then you can add secret config thought it's name which is it's reference like the example below
    
    ```
      containers:
      - name: django-kub8
        image: registry.digitalocean.com/private-registery/django-kub8:latest
      * envFrom:
      *   - secretRef:
      *       name: django-kub8-prod-env
    ```
    
    3- finally, you can apply changes 
    $ kubectl apply -f k8s/apps/django-k8s-web.yaml
    

#### -- To get into a pod is deployed container :
    
    $ kubectl get pods
    django-kub8-deployment-6d4fd679f8-8q469
    
    $ kubectl exec -it django-kub8-deployment-6d4fd679f8-8q469 -- /bin/bash

    
#### -- Logs & Troubleshooting Kubernetes  

    $ kubectl logs <pod_name>

#### -- Make a Github Actions workflow job depends on other one :
    
    '''
    jobs:
      django_testing_job:
        uses: MDRCS/django-kub8/.github/workflows/ci.yaml@main
      build:
        runs-on: ubuntu-latest
        needs: [django_testing_job]
    '''

    + for the `build` to start running we should have validated `django_testing_job` ci pipeline successfuly.

#### -- Create Env file in the ci pipeline environment and create secret based on it :

    ```
     - name: Update deployment secrets
      run: |
        cat << EOF >> web/.env.prod
        DJANGO_SUPERUSER_USERNAME=${{ secrets.DJANGO_SUPERUSER_USERNAME }}
        DJANGO_SUPERUSER_PASSWORD=${{ secrets.DJANGO_SUPERUSER_PASSWORD }}
        DJANGO_SUERPUSER_EMAIL=${{ secrets.DJANGO_SUERPUSER_EMAIL }}
        DJANGO_SECRET_KEY=${{ secrets.DJANGO_SECRET_KEY }}
        ENV_ALLOWED_HOST=${{ secrets.ENV_ALLOWED_HOST }}
        POSTGRES_DB=${{ secrets.POSTGRES_DB }}
        POSTGRES_PASSWORD=${{ secrets.POSTGRES_PASSWORD }}
        POSTGRES_USER=${{ secrets.POSTGRES_USER }}
        POSTGRES_HOST=${{ secrets.POSTGRES_HOST }}
        POSTGRES_PORT=${{ secrets.POSTGRES_PORT }}
        EOF
        kubectl delete secret django-kub8-prod-env
        kubectl create secret generic django-kub8-prod-env --from-env-file=web/.env.prod
    ```

    + All secrets should be stored in github secrets.
      Ref: https://github.com/MDRCS/django-kub8/settings/secrets/actions

