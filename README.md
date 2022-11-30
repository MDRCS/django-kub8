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
s
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