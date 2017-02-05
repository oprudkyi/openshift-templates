Memcached template
==============

This is a template for memcached service

 - uses memcached image from [official memcached repository at docker hub](https://hub.docker.com/r/_/memcached/)
 - allows to setup cache size 
 - autoupdate is enabled 

Usage
==============

```
oc login -u system:admin
oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/memcached/memcached.yaml -n openshift

```

It will be added under 'Data Stores' Group in openshift catalog of templates
