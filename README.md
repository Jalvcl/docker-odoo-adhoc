odoo-adhoc
==========

Odoo official dockerfile: https://github.com/odoo/docker/blob/master/8.0/Dockerfile
Odoo official image: https://registry.hub.docker.com/_/odoo/
Odoo official docker documentation: https://github.com/docker-library/docs/tree/master/odoo

This is an odoo image extended from the official one but with more dependencies installed.

Configuration
-------------

Docker configuration for Odoo

PostgreSQL service
------------------
Launch the PostgreSQL service:
    docker run --rm -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo --name db-[instance_complete_name] -v /opt/odoo/[env_name]/[instance_name]/postgresql:/var/lib/postgresql/data postgres:9.3

eg.
    docker run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo --name db-logos-80-main -v /opt/odoo/logos-80/main/postgresql:/var/lib/postgresql/data postgres:9.4

or with a disposable container:
    docker run --rm -ti -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo --name db-logos-80-main -v /opt/odoo/logos-80/main/postgresql:/var/lib/postgresql/data postgres:9.4

Odoo service
------------
You can launch odoo connected to postgres with the following command:
    docker run -p 127.0.0.1:[instance_xml_port]:8069 -p 127.0.0.1:[instance_longp_port]:8072 --name odoo-[instance_complete_name] --link db-[instance_complete_name]:db -t adhoc/odoo-adhoc

Options:
    - Custom conf file located in /opt/odoo/[env_name]/[instance_name]/config/openerp-server.conf
        -v /opt/odoo/[env_name]/[instance_name]/config:/etc/odoo
    - Custom addons path:
        -v /opt/odoo/[env_name]/[instance_name]/addons:/mnt/extra-addons
    - Custom data dir:
        -v /opt/odoo/[env_name]/[instance_name]/data_dir:/var/lib/odoo
    - Especificar argumentos, agregar al final:
        -- --dbfilter=odoo_db_.*
        Tipicamente:
            -- --workers=9

eg.
    docker run -p 127.0.0.1:8069:8069 -p 127.0.0.1:8072:8072 --name odoo-logos-80-main --link db-logos-80-main:db -t -v /opt/odoo/logos-80/main/data_dir:/var/lib/odoo adhoc/odoo-adhoc

or with a disposable container:
    
    ## Sin Nada especial
    docker run --rm -ti -p 127.0.0.1:8069:8069 -p 127.0.0.1:8072:8072 --name odoo-logos-80-main --link db-logos-80-main:db -t adhoc/odoo-adhoc

    ## Con custom data_dir
    docker run --rm -ti -p 127.0.0.1:8069:8069 -p 127.0.0.1:8072:8072 --name odoo-logos-80-main --link db-logos-80-main:db -t -v /opt/odoo/logos-80/main/data_dir:/var/lib/odoo adhoc/odoo-adhoc
    
    ## Con custom addons 
    docker run --rm -ti -p 127.0.0.1:8069:8069 -p 127.0.0.1:8072:8072 --name odoo-logos-80-main --link db-logos-80-main:db -t -v /opt/odoo/logos-80/main/sources:/mnt/extra-addons adhoc/odoo-adhoc
    
    ## Con custom config
    docker run --rm -ti -p 127.0.0.1:8069:8069 -p 127.0.0.1:8072:8072 --name odoo-logos-80-main --link db-logos-80-main:db -t -v /opt/odoo/logos-80/main/config:/etc/odoo adhoc/odoo-adhoc
    
    ## Con custom config y custom addons path
    docker run --rm -ti -p 127.0.0.1:8069:8069 -p 127.0.0.1:8072:8072 --name odoo-logos-80-main --link db-logos-80-main:db -t -v /opt/odoo/logos-80/main/config:/etc/odoo -v /opt/odoo/logos-80/main/sources:/mnt/extra-addons adhoc/odoo-adhoc
    
    ## Con custom config, custom addons path y custom data dir
    docker run --rm -ti -p 127.0.0.1:8069:8069 -p 127.0.0.1:8072:8072 --name odoo-logos-80-main --link db-logos-80-main:db -t -v /opt/odoo/logos-80/main/config:/etc/odoo -v /opt/odoo/logos-80/main/sources:/mnt/extra-addons -v /opt/odoo/logos-80/main/data_dir:/var/lib/odoo adhoc/odoo-adhoc

Start and Stop an Odoo Instance
-------------------------------
docker stop odoo
docker start -a odoo

Stop and restart a PostgreSQL server
------------------------------------
When a PostgreSQL server is restarted, the Odoo instances linked to that server must be restarted as well because the server address has changed and the link is thus broken.

Development
-----------
  Podemos agregar /bin/bash
  Luego correr
  runuser -u odoo /usr/bin/openerp-server -- --config=/etc/odoo/openerp-server.conf

If you want to use this docker image for development, you can launch it using:

    sudo docker run --rm -ti --name odoo --link postgres:odoo-db \
      -v /your/local/repo:/opt/odoo/sources/addons \
      -p 8069:8069 adhoc/odoo:8.0

where /your/local/repo is your currently cloned repository of some module. This will allow you to change the code of the module locally without the need of creating an image every time you modify the module.

If you want to update all modules in the database "database_to_update" then you can pass the complete command when you launch the container this way:

    sudo docker run --rm -ti --name odoo --link postgres:odoo-db \
      -v /your/local/repo:/opt/odoo/sources/addons \
      -p 8069:8069 adhoc/odoo:8.0 \
      sudo -H -u odoo /opt/odoo/server/odoo.py -c /opt/odoo/server/odoo.conf \
      --update=all -d database_to_update

If you also want to use a local configuration for odoo you can use volumes as well with the following option:

    sudo docker run --rm -ti --name odoo --link postgres:odoo-db \
      -v /your/local/repo:/opt/odoo/sources/addons \
      -v /your/local/odoo.conf:/opt/odoo/odoo.conf \
      -p 8069:8069 adhoc/odoo:8.0 \
      sudo -H -u odoo /opt/odoo/server/odoo.py -c /opt/odoo/odoo.conf \
      --update=all -d database_to_update
