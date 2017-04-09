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


License
==============

OpenShift Origin template for Nginx is licensed under [The MIT License (MIT)](LICENSE).

