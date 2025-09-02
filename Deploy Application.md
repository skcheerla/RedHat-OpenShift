# How to Deploy Application on OpenShift via CLI 

Commands --
oc new-project chapter1
oc new-app --name hello --image quay.io/redhattraining/hello-world-nginx:v1.0
oc expose service hello
oc get all
oc get pod -o wide
oc scale --replicas 4 deployment/hello
oc delete project chapter1
