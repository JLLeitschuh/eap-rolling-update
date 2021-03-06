# Kubernetes Rolling Update Demo
This example app leverages the functionality of Kubernetes and OpenShift projects to run enterprise software in distributed environment.
Main goal is to demonstrate the application architecture - an application cluster that can be scaled and cluster nodes can survive restart.

```
# add --insecure-registry 172.30.0.0/16 to /etc/sysconfig/docker
oc cluster up --version=latest

# create git infrastructure
oc new-project git --display-name="GIT"
oc process -f https://raw.githubusercontent.com/josefkarasek/eap-rolling-update/master/gogs-postgresql.json | oc create -f -
oc deploy dc/postgresql-gogs --latest
oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default

# create the app cluster
oc new-project cluster1 --display-name="CLUSTER"

# service account with secret
oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/secrets/eap7-app-secret.json

# grant view rights on the project
oc policy add-role-to-user view system:serviceaccount:$(oc project -q):eap7-service-account -n $(oc project -q)

# image streams
oc create -f https://raw.githubusercontent.com/josefkarasek/eap-rolling-update/master/eap70-is.json
oc create -f https://raw.githubusercontent.com/josefkarasek/eap-rolling-update/master/postgres-is.json

# template
oc create -f https://raw.githubusercontent.com/josefkarasek/eap-rolling-update/master/eap70-postgresql-demo-s2i.json

# build and deploy
# substitute SOURCE_REPOSITORY_URL with your gogs project URL
oc new-app --template=eap70-postgresql-demo-s2i \
  -p SOURCE_REPOSITORY_URL=https://github.com/josefkarasek/eap-rolling-update \
  -p SOURCE_REPOSITORY_REF=master \
  -p CONTEXT_DIR=greeter \
  -p DB_JNDI=java:jboss/datasources/GreeterQuickstartDS \
  -p DB_DATABASE=USERS \
  -p HTTPS_NAME=jboss \
  -p HTTPS_PASSWORD=mykeystorepass \
  -p JGROUPS_ENCRYPT_NAME=secret-key \
  -p JGROUPS_ENCRYPT_PASSWORD=password \
  -p IMAGE_STREAM_NAMESPACE=cluster1 \
  -p NODES=3
```

### Acknowledgements
[Gogs](https://gogs.io/) image based on [Erik Jacobs' image.](https://github.com/OpenShiftDemos/gogs-openshift-docker)
Greeter demo application based on [EAP 7 quickstarts](https://github.com/jboss-developer/jboss-eap-quickstarts), distributed cache added by [Jiri Pechanec](https://github.com/jpechane/jboss-eap-quickstarts/tree/7.0.x-develop/greeter)
