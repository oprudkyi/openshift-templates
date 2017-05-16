[OpenShift Origin](https://www.openshift.org/) template for [GitLab Runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner)
==============

This is a template for easy deployment of GitLab Runner CI into OpenShift cluster (Openshift Online isn't supported) 

 - uses official [GitLab Runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner) image from [gitlab docker repo](https://hub.docker.com/r/gitlab/gitlab-runner/) 
 - caching is implemented via official [Minio Cloud Storage](https://www.minio.io/) image from [minio docker repo](https://hub.docker.com/r/minio/minio/)
 - provides sane default options and simple configurator
 - containers are runnning with `anyuid` SCC (allows to create new docker containers for CI and run them as root)
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

 - [OpenShift Origin](https://www.openshift.org/) (The template is tested with [All-in-one cluster binary](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md), it won't work on the [Openshift Online](https://www.openshift.com/) 
 - admin access to server for setting rights for service account 


Installation
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

3. Setup Security Context Constraints (SCC) for service accounts used for running containers (`anyuid` means commands inside containers can run as root)

	```sh
	oc login -u system:admin
	oc adm policy add-scc-to-user anyuid -z sa-gitlab-runner -n prj-gitlab-runner
	oc adm policy add-scc-to-user anyuid -z sa-minio -n prj-gitlab-runner
	```

4. Go to web console https://MASTER-IP:8443/console/project/prj-gitlab-runner/overview (where MASTER-IP is IP where cluster is bound) and press "Add to Project" and select "gitlab-runner" template

5. Fill required fields
  - GitLab Runner Token : one from `/etc/gitlab-runner/config.toml`
  - GitLab Runners Namespace : `prj-gitlab-runner`

6. As well there are some additional options you may configure - docker hub tags for GitLab-Runner and Minio, login/password for Minio etc, though defaults will work as well

7. After pressing update the deployment will start, it may take few minutes to download required images and preconfigure them 

8. In your Gitlab Project check "Runners" page to have runner activated 

9. Run some CI job , there will be something like 

	```
	Waiting for pod prj-gitlab-runner/runner-86251ae3-project-1142978-concurrent-0uzqax to be running, status is Pending
	```
	in log output of CI 


Persistent cache in directory of your host (optional)
==============

Minio server is not attached to any permanent storage and uses an ephemeral storage - [emptyDir](http://kubernetes.io/docs/user-guide/volumes/#emptydir). When Minio Service/Pod is stopped or restarted all data will be deleted. 
Though, while Minio is running, cache is available locally via some path like '/var/lib/origin/openshift.local.volumes/pods/de1d0ff7-d2bb-11e6-8d5b-74d02b8fa488/volumes/kubernetes.io~empty-dir/vol-minio-data-store'

So, you may need to point `vol-minio-data-store` volume to persistent storage or periodically backup data.

While you can use any storage - NFC/Ceph RDB/GlusterFS and [more](https://docs.openshift.org/latest/install_config/persistent_storage/index.html), 
for simple cluster setup (with small number of nodes) host path is the simplest. Though if you have more then one Node you should mantain cleanup/sync between nodes by self. 

Next steps allow to use local directory `/cache/gitlab-runner` as storage for Minio 

1. Setup Security Context Constraints (SCC) for Minio container to access Node's filesystem

	```sh
	oc login -u system:admin
	oc adm policy add-scc-to-user hostmount-anyuid -z sa-minio -n prj-gitlab-runner
	```
2. Edit `dc-minio-service` deployment config via OpenSift Web console 
at https://MASTER-IP:8443/console/project/prj-gitlab-runner/edit/yaml?kind=DeploymentConfig&name=dc-minio-service 
or from console 
	```sh
	oc project prj-gitlab-runner
	oc edit dc/dc-minio-service
	```

  Replace 
  ```
      volumes:
      - emptyDir: {}
        name: vol-minio-data-store
  ```

  with 
  ```
      volumes:
      - hostPath: 
          path: /cache/gitlab-runner
        name: vol-minio-data-store
  ```

  After saving, Minio server will be automatically restarted and you can access cache via 
  Minio Web console http://minio-service.prj-gitlab-runner.svc.cluster.local/minio/bkt-gitlab-runner/, 
  you can try to upload file and check if it exists at the `/cache/gitlab-runner`
  as well you can force new deploy (restart) of minio and see if it keeps files on restart


Management
==============

- You can additionally configure gitlab runner via web console at https://MASTER-IP:8443/console/project/prj-gitlab-runner/browse/config-maps/cm-gitlab-runner , by example count of concurent jobs etc, see all possible options at GitLab Runner [docs](https://docs.gitlab.com/runner/configuration/advanced-configuration.html).

	Alternatively you can use console for editing:
	```sh
	oc project prj-gitlab-runner
	oc edit configmap/cm-gitlab-runner
	```

	After editing you will need to manually "Deploy" gitlab-runner deployment - https://MASTER-IP:8443/console/project/prj-gitlab-runner/browse/dc/dc-gitlab-runner-service or via console 
	```sh
	oc project prj-gitlab-runner
	oc deploy dc-gitlab-runner-service --latest --follow=true
	```

- Minio Web console is available at http://minio-service.prj-gitlab-runner.svc.cluster.local/ or just grab IP under https://MASTER-IP:8443/console/project/prj-gitlab-runner/browse/services/minio-service and access/secret keys under https://MASTER-IP:8443/console/project/prj-gitlab-runner/browse/dc/dc-minio-service?tab=environment

License
==============

OpenShift Origin template for GitLab Runner is licensed under [The MIT License (MIT)](LICENSE).

[Blog](http://oleksii-prudkyi.blogspot.com/2017/01/openshift-origin-template-for-gitlab-runner.html)
