[OpenShift Origin](https://www.openshift.org/) template for MySQL(or Percona/MariaDB) images from official repositories at docker hub.
==============

This is a template for easy deployment of official MySQL (or forks) for small (All-in-one) or middle-sized Openshift deployments

 - Uses official images from [MySQL](https://hub.docker.com/r/library/mysql/), [Percona](https://hub.docker.com/r/library/percona/) or 
 [MariaDB](https://hub.docker.com/r/library/mariadb/) repositories
 - Allows to select docker image tags (versions)
 - Uses local fs on node (hostmount SCC) to store files (you will need to edit template to use persistent volumes)
 - Allows to configure my.cnf on creating service from template or later (via ConfigMap)
      
Based on mysql-ephemeral-template.yml


Usage
==============

1. Import template

	Globally:
	```sh
	oc login -u system:admin
	oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/hostmount-official-mysql-percona-mariadb/hostmount-official-mysql-percona-mariadb.yaml -n openshift
	```

	or just into your project
	```sh
	oc login -u developer
	oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/hostmount-official-mysql-percona-mariadb/hostmount-official-mysql-percona-mariadb.yaml -n CURRENT_PROJECT_NAME
	```

2. Setup Security Context Constraints (SCC) for service accounts used for running containers 
(`anyuid` means commands inside containers can run as root). Please use your project name instead of `CURRENT_PROJECT_NAME`

	```sh
	oc login -u system:admin
	oc adm policy add-scc-to-user hostmount-anyuid -z sa-hostmount-mysql -n CURRENT_PROJECT_NAME
	```


It will be available under 'Data Stores' Group in openshift catalog of templates


License
==============

OpenShift Origin template for MySQL is licensed under [The MIT License (MIT)](LICENSE).


