[OpenShift Origin](https://www.openshift.org/) template for PHP-FPM images from official repositories at docker hub.
==============

This is a generic template, that moves PHP-FPM configuration file (/usr/local/etc/php-fpm.conf) into ConfigMap, you will need to configure it 
for your needs via ConfigMap and by attaching persistent volumes with php files.

 - Uses official images from [php](https://hub.docker.com/r/library/php/) repository
 - Allows to select docker image tags (versions)
 - Allows to configure php-fpm.conf and other confirurations via ConfigMap
      

Usage
==============

1. Import template

	for all projects:
	```sh
	oc login -u system:admin
	oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/php-fpm/php-fpm.yaml -n openshift
	```

	or just into your project
	```sh
	oc login -u developer
	oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/php-fpm/php-fpm.yaml -n CURRENT_PROJECT_NAME
	```

It will be available under 'PHP' Group in openshift catalog of templates

Configuration 
==============
1. Attach storage to deployment and map it to some folder, by example /var/www/path-inside-container
2. Configure your web server to use /var/www/path-inside-container as root path for server and forward requests to this service.
  i.e. for [nginx](https://gitlab.com/oprudkyi/openshift-templates/tree/master/nginx) :
  ```
  oc edit cm/cm-nginx-nginx-config
  ```
  and add something like

  ```
		location / {
			try_files $uri $uri/ /index.php$is_args$args;
		}

		location ~ \.php$ {
            root   /var/www/path-inside-container;
			try_files $uri /index.php =404;
			fastcgi_pass php-fpm:9000;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include fastcgi.conf;
			#fastcgi_buffers 4 512k; 
			#fastcgi_buffer_size 512k; 
			#fastcgi_busy_buffers_size 512k; 
			...
        }
  ```


Hostmount (optional)
==============
1. Add Security Context Constraints (SCC) for Nginx container to access Node's filesystem
	```sh
	oc login -u system:admin
	oc adm policy add-scc-to-user hostaccess -z sa-php-fpm -n CURRENT_PROJECT_NAME
	```
2. Edit `dc-php-fpm` deployment config via OpenSift Web console 
at https://MASTER-IP:8443/console/project/CURRENT-PROJECT-NAME/edit/yaml?kind=DeploymentConfig&name=dc-php-fpm 
or from console 
	```sh
	oc edit dc/dc-php-fpm
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
          path: /path-inside-opeshift-node(host)
        name: vol-my-site
  ```

  Then you can configure your web server to use /var/www/path-inside-container as root path for server and forward requests to this service.
  i.e. for nginx :
  ```
  oc edit cm/cm-nginx-nginx-config
  ```
  and have something like
  ```
		location / {
			try_files $uri $uri/ /index.php$is_args$args;
		}

		location ~ \.php$ {
            root   /var/www/path-inside-container;
			try_files $uri /index.php =404;
			fastcgi_pass php-fpm:9000;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include fastcgi.conf;
			#fastcgi_buffers 4 512k; 
			#fastcgi_buffer_size 512k; 
			#fastcgi_busy_buffers_size 512k; 
			...
        }

  ```




License
==============

OpenShift Origin template for PHP-FPM is licensed under [The MIT License (MIT)](LICENSE).


