# django-caprover-notes
Some notes regarding Django (and CapRover)

## Awesome database

````python
DATABASES = {
    'default': dj_database_url.config(default=os.environ['DATABASE_URL'])
}
````

## Thumbnails with SORL thumbnails + S3

You will have performance issues unless you do the following:

````python
AWS_S3_ENDPOINT_URL = 'xx'
AWS_S3_CUSTOM_DOMAIN  = 'xx'
AWS_S3_URL_PROTOCOL = 'https'
AWS_QUERYSTRING_AUTH = False
````

````python
from sorl.thumbnail.admin import AdminImageMixin


class PlaceAdmin(AdminImageMixin, admin.ModelAdmin):
    pass

admin.site.register(Place, PlaceAdmin)
````

````html
{% thumbnail place.image "800x500" crop="center" as im %}
  <a href="{{ place.image.url }}"><img class="card-img-top" src="{{ im.url }}" alt="Image of {{ place.name }}"></a>
{% endthumbnail %}
````

## Static stuff

````python
STATIC_ROOT = os.path.join(BASE_DIR, "static")
STATIC_URL = '/static/'
````

## Security

````python
ALLOWED_HOSTS = ['blablaba.kirjaimet.xyz']
SECURE_HSTS_SECONDS = 3600 * 24 * 365
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
````

## Internal IP + Docker

````python
if not IS_PRODUCTION:
    import socket

    # tricks to have debug toolbar when developing with docker
    ip = socket.gethostbyname(socket.gethostname())
    INTERNAL_IPS += [ip[:-1] + '1']
````

## Deploying with CapRover

````docker
FROM library/python:3.8
RUN apt-get update && apt-get upgrade -y && apt-get autoremove && apt-get autoclean
RUN apt-get install -y \
    libffi-dev \
    libssl-dev \
    libxml2-dev \
    libxslt-dev \
    libjpeg-dev \
    libfreetype6-dev \
    zlib1g-dev \
    net-tools \
    vim
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY ./Pipfile* /usr/src/app/
RUN pip install -U pipenv
RUN pipenv install --system --dev
COPY ./ /usr/src/app
EXPOSE 80
CMD python manage.py collectstatic --noinput && python manage.py migrate && gunicorn helsinki.wsgi --bind=0.0.0.0:80
````
