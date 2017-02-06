MailCatcher template
==============

This is a template for [MailCatcher service](https://mailcatcher.me/)

MailCatcher runs a super simple SMTP server which catches any message sent to it to display in a web interface. 
Add mailcatcher to your OpenShift Origin project, set your favourite app to deliver to smtp://mailcatcher:25 instead of your default SMTP server, 
then check out http://mailcatcher.PROJECT-NAME.svc.cluster.local/ to see the mail that's arrived so far.


 - uses mailcatcher image from [unofficial mailcatcher repository at docker hub](https://hub.docker.com/r/schickling/mailcatcher/)
 - autoupdate is enabled 

Usage
==============

```
oc login -u system:admin
oc replace --force -f https://gitlab.com/oprudkyi/openshift-templates/raw/master/mailcatcher/mailcatcher.yaml -n openshift

```

It will be added under 'Uncategorized' Group in openshift catalog of templates

Please note, by default there no Route for MailCatcher because it doesn't support any security (i.e. it won't be available for external access). 

After adding to project MailCatcher's Web GUI will be available via http://mailcatcher.PROJECT-NAME.svc.cluster.local/ or what domain you have for OpenShift or grab IP via

```
oc get services -n PROJECT-NAME | grep mailcatcher
```

SMTP server is available inside project via smtp://mailcatcher:25 (you have to setup email Host/Port properly in your Application)  

