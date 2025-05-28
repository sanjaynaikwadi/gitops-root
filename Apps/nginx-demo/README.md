# Nginx StatefulSet

Yaml file will create service and statefulset of nginx, this can be run simply on Vanilla k8s cluster

Make sure update the StorageClass in the yaml file before applying the yaml file

If you are using Openshift then you need to follow the additional steps 
```file
oc create ns nginx-demo
oc adm policy add-scc-to-user anyuid -z default -n nginx-demo

```

