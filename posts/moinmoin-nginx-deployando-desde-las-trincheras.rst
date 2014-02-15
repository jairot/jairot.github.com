.. link: 
.. description: 
.. tags: 
.. date: 2014/02/14 15:52:16
.. title: MoinMoin + Supervisor + Nginx (O como deployar tu propia wiki)
.. slug: moinmoin-nginx-deployando-desde-las-trincheras

Las Wikis son como las Tablets, todo el mundo quiere una pero nadie sabe bien para que sirven. Por esta misma razon es que las probabilidades de que en algun momento te toque deployar una wiki son bastante altas. 

El mundillo de las wikis es bastante `amplio <http://en.wikipedia.org/wiki/Comparison_of_wiki_software>`_, pero es este post voy a detallar los pasos para deployar de una manera bastante "Quick-and-dirty" una instancia de `MoinMoin  <http://moinmo.in/>`_,  usando Nginx, Supervisor y el servidor Standalone que MoinMoin trae incorporado. Mi intención inicial era usar guinicorn pero no encontre una manera simple de hacerlo (Si alguien sabe como que avise!).

Instalando y configurando MoinMoin
----------------------

Como MoinMoin esta hecho en python vamos a asegurarnos que tenemos instaladas todas las herramientas de entorno necesarias para trabajar

::

    $ sudo apt-get install python-pip python-virtualenv supervisor nginx

como las normas de higiene y seguridad indican, creamos un virtualenv y una vez dentro del mismo instalamos MoinMoin

::
    
    $ virtualenv miwiki
    $ cd miwiki/
    $ source bin/activate
    $ pip install moin

MoinMoin trae una configuración por defecto, pero necesitamos poner los archivos de configuración en un lugar accesible

::
    
    $ mkdir setup
    $ cp -r share/moin/config/ setup/
    $ cp -r share/moin/data/ setup/config/data
    $ cp -r share/moin/underlay/ setup/config/underlay

Ahora ejecutamos el server Standalone

::
    
    moin --config-dir=setup/config server standalone

Abrimos un browser, tecleamos localhost:8080 en la barra del navegador y nos encontraremos con la pagina de configuración de MoinMoin. Seguimos los pasos que nos indica y ya casi estamos. 

El próximo paso es hacer un script para que Supervisor ejecute, para esto escribimos lo siguiente en un archivo llamado start.sh

::
    #!/bin/bash
     
    NAME="wiki" # Name of the application
    DIR=/home/serveruser/miwiki/ #project directory
    USER=serveruser  #the user to run as
    GROUP=serveruser
     
     
    echo "Starting $NAME"
     
    # Activate the virtual environment
    cd $DIR
    source bin/activate
    #export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
    export PYTHONPATH=$DIR:$PYTHONPATH
     
    # Create the run directory if it doesn't exist
    # Programs meant to be run under supervisor should not daemonize themselves (do not use --daemon)
    moin --config-dir=setup/config server standalone --port=4002

Para confirmar que todo esta bien, corremos start.sh y comprobamos que MoinMoin funciona (ahora en el puerto 4002).

Ahora resta indicarle a Supervisor que se acuerde de mantener levantado el servidor de nuestra wiki, creamos un archivo en /etc/supervisor/conf.d/wiki.conf

::

    [program:wiki]
    command = /home/serveruser/miwiki/start.sh ; Command to start app
    user = serveruser ; User to run as
    stdout_logfile = /home/serveruser/miwiki/log.log ;
    redirect_stderr = true ; Save stderr in the same log

Guardamos los cambios y corremos los comandos necesarios para que Supervisor levante el servidor de la wiki

::
    $ sudo supervisorctl reread all
    $ sudo supervisorctl restart all

Ya estamos casi listos! Solo falta un ultimo detalle, configurar nginx para que sirva MoinMoin, para esto creamos un archivo en /etc/nginx/sites-available/wiki.confcon los siguiente datos

::
    server {

        listen 80;
        server_name wiki.mydomain.com;

        client_max_body_size 4G;

        access_log /home/serveruser/miwiki/nginx-access.log;
        error_log /home/serveruser/miwiki/nginx-error.log;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_redirect off;

            if (!-f $request_filename) {
                proxy_pass http://127.0.0.1:4002;
                break;
              }
         }

      }

(Nota: Es posible que este archivo de configuración no sea de lo mejor, se muy poco de nginx y seguro hay muchas cosas para mejorarle, pero asi funciona)

Para terminar habilitamos el sitio creando un enlace simbólico en sites-enable y reiniciamos Nginx

::
    $ sudo ln /etc/nginx/sites-available/wiki.conf /etc/nginx/sites-enable/wiki.conf
    $ sudo service nginx restart

Listo el pollo! Ahora deberíamos ver la wiki en wiki.mydomain.com

Fácil no? Estoy seguro que muchos de ustedes tienen optimizaciones y/o correcciones para esto! Si es así les agradecería que me las envíen a cualquiera de los medios de contacto que están por ahí!

Saludos, J


