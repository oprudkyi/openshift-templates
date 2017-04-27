[OpenShift Origin](https://www.openshift.org/) template for Nginx images from official repositories at docker hub.
==============

This is a generic template, that moves Nginx configuration files (nginx.conf etc) into ConfigMap, you will need to configure it 
for your needs via ConfigMap and by attaching presistent volumes with static content or forwarding requests to fastcgi services.

 - Uses official images from [Nginx](https://hub.docker.com/r/library/nginx/) repository
 - Allows to select docker image tags (versions)
 - Allows to configure nginx.conf and other confirurations via ConfigMap
      

Usage
==============

1. Import template

	for all projects:
	```sh
	oc login -u system:admin
	oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/nginx/nginx.yaml -n openshift
	```

	or just into your project
	```sh
	oc login -u developer
	oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/nginx/nginx.yaml -n CURRENT_PROJECT_NAME
	```

It will be available under 'Uncategorized' Group in openshift catalog of templates

Notes
==============
With default settings container serve static content from /usr/share/nginx/html from image. 
You can replace it with your volumes or configure ConfigMap data to forward requests to other services (fastcgi/php-fpm etc)

Nginx is run under non-priveledged user account. So, when porting your config please note:
 - container can't listen on priveledged TCP ports (<1024), by default used 8080 port
 - nginx can't write to many folder on image, i.e. ```pid```/```client_body_temp_path``` should use /tmp instead 

Configuration 
==============
1. Attach storage to deployment and map it to some folder, by example /var/www/path-inside-container. 
2. Configure your web server to use /var/www/path-inside-container as root path.
  ```
  oc edit cm/cm-nginx-nginx-config
  ```

  and add something like

  ```
        location / {
            root   /var/www/path-inside-container;
			...
        }

  ```

Hostmount (optional)
==============
1. Add Security Context Constraints (SCC) for Nginx container to access Node's filesystem
	```sh
	oc login -u system:admin
	oc adm policy add-scc-to-user hostaccess -z sa-nginx -n CURRENT_PROJECT_NAME
	```
2. Edit `dc-nginx` deployment config via OpenSift Web console 
at https://MASTER-IP:8443/console/project/CURRENT_PROJECT_NAME/edit/yaml?kind=DeploymentConfig&name=dc-nginx 
or from console 
	```sh
	oc edit dc/dc-nginx
	```

  Add, under volumeMounts new mountPath, i.e.
  ```
        volumeMounts:
        - mountPath: /var/www/path-inside-container
          name: vol-my-site
  ```

  And, under volumes, 
  ```
      volumes:
      - hostPath: 
          path: /path-inside-opeshift-node
        name: vol-my-site
  ```

  Then you can edit nginx config map and configure it to use /var/www/path-inside-container as root path for server.
  ```
  oc edit cm/cm-nginx-nginx-config
  ```
  and have something like
  ```
        location / {
            root   /var/www/path-inside-container;
			...
        }

  ```

PHP
==============

Basic PHP site.

Please note, advanced php frameworks (Laravel) tends to cache full paths, so, if you are planning to use command 
tools from host and from container simultaneously (by example run composer/artisan on host node), then you should use the same path inside containers (php,nginx) and hostnode. 


Configure ConfigMap (default.conf) 

  ```
  oc edit cm/cm-nginx-nginx-config
  ```

  Replace it with 
  ```
    upstream php-upstream { 
        server php:9000; 
    }
    server {

        listen 8080 default_server;
        listen [::]:8080 default_server ipv6only=on;

        server_name locahost;
        root /var/www/path-inside-container/public;
        index index.php index.html index.htm;
    
        location / {
             try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
            try_files $uri /index.php =404;
            fastcgi_pass php-upstream;
            fastcgi_index index.php;
            fastcgi_buffers 16 16k; 
            fastcgi_buffer_size 32k;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include /etc/nginx/fastcgi_params;
        }

        location ~ /\.ht {
            deny all;
        }
    }
  ```

Setup [PHP-FPM image](https://gitlab.com/oprudkyi/openshift-templates/tree/master/php-fpm)   



License
==============

OpenShift Origin template for Nginx is licensed under [The MIT License (MIT)](LICENSE).

