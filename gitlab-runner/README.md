[OpenShift Origin](https://www.openshift.org/) template for [GitLab Runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner)
==============

This is a template for easy deployment of GitLab Runner CI into OpenShift cluster 

 - uses official [GitLab Runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner) image from [gitlab docker repo](https://hub.docker.com/r/gitlab/gitlab-runner/) 
 - caching is implemented via official [Minio Cloud Storage](https://www.minio.io/) image from [minio docker repo](https://hub.docker.com/r/minio/minio/)
 - provides sane default options and simple configurator
 - containers are run with admin rights and anyuid SCC (allows to create new docker containers for CI and run them as root)
 - autoupdate is enabled 
 - partially is based on kubernetes configs 
	
	- https://docs.gitlab.com/runner/install/kubernetes.html
	- https://gitlab.com/gitlab-org/omnibus-gitlab/blob/ux-demo/docker/openshift-ci-template.json
	- http://blog.lwolf.org/post/how-to-easily-deploy-gitlab-on-kubernetes/
	- https://github.com/lwolf/kubernetes-gitlab/tree/master/minio

Prerequisites 
==============

 - GitLab Runner's local token, from `/etc/gitlab-runner/config.toml`. That file is created after 
 [installation and registration of GitLab Runner](https://docs.gitlab.com/runner/install/)
 and after running of 
	```sh
	sudo gitlab-ci-multi-runner register
	```
 there will be file `/etc/gitlab-runner/config.toml` with token 
 ```
[[runners]]
	...
	token = "..."
 ```

 - [OpenShift Origin](https://www.openshift.org/) (The template is tested with [All-in-one cluster binary](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md), thought it might work with [Red Hat OpenShift](https://www.openshift.com/) as well
 - admin access to server for setting rights for service account 


Usage
==============

1. Create new project/namespace 

	```sh
	oc login -u developer
	oc new-project prj-gitlab-runner
	```

2. Import template

	```sh
	oc create -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/gitlab-runner/gitlab-runner.yaml -n prj-gitlab-runner
	```

3. Create Service Account and give him admin role in project

	```sh
	oc create serviceaccount sa-gitlab-runner -n prj-gitlab-runner
	oc adm policy add-role-to-user admin -z sa-gitlab-runner -n prj-gitlab-runner 
	```

4. Give him ability to run as root inside containers 

	```sh
	oc login -u system:admin
	oc adm policy add-scc-to-user anyuid -z sa-gitlab-runner -n prj-gitlab-runner
	```

5. Go to web console https://MASTER_IP:8443/console/project/prj-gitlab-runner/overview (where MASTER_IP is IP where cluster is bound) and press "Add to Project" and select "gitlab-runner" template

6. Fill required fields

  - GitLab Runner Token : one from `/etc/gitlab-runner/config.toml`
  - GitLab Runners Namespace : `prj-gitlab-runner`

7. As well there are some additional options you may configure - docker hub tags for GitLab-Runner and Minio, login/password for Minio etc, though defaults will work as well

8. After pressing update the deployment will start, it may take few minutes to download required images and preconfigure them 

9. In your Gitlab Project check "Runners" page to have runner activated 

10. Run some CI job , there will be something like 

	```
	Waiting for pod prj-gitlab-runner/runner-86251ae3-project-1142978-concurrent-0uzqax to be running, status is Pending
	```
	in log output of CI 

Management
==============

- You can additionally configure gitlab runner via web console at https://MASTER_IP:8443/console/project/prj-gitlab-runner/browse/config-maps/cm-gitlab-runner , by example count of concurent jobs etc, see all possible options at GitLab Runner [docs](https://docs.gitlab.com/runner/configuration/advanced-configuration.html).

	After editing you will need to manually "Deploy" gitlab-runner deployment - https://MASTER_IP:8443/console/project/prj-gitlab-runner/browse/dc/dc-gitlab-runner-service

- Minio Web console is available at http://minio-service.prj-gitlab-runner.svc.cluster.local/ or just grab IP under https://MASTER_IP:8443/console/project/prj-gitlab-runner/browse/services/minio-service and access/secret keys under https://MASTER_IP:8443/console/project/prj-gitlab-runner/browse/dc/dc-minio-service?tab=environment

- Minio server is not attached to any permanent storage and all data will be lost on restart of Pod, you may need to point `vol-minio-data-store volume` to permanent storage or periodically backup data 
(it is stored locally under some path like '/var/lib/origin/openshift.local.volumes/pods/de1d0ff7-d2bb-11e6-8d5b-74d02b8fa488/volumes/kubernetes.io~empty-dir/vol-minio-data-store')

	

