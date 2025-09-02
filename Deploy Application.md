# How to Deploy Application on OpenShift via CLI 

## Commands --  

oc new-project chapter1  

oc new-app --name hello --image quay.io/redhattraining/hello-world-nginx:v1.0  


<img width="832" height="357" alt="image" src="https://github.com/user-attachments/assets/b4bdde73-9e90-4edc-82a8-d2aaaa19c748" />

oc get all  

<img width="837" height="290" alt="image" src="https://github.com/user-attachments/assets/67291901-6f0c-41bf-aff0-3f6fcebfb1d1" />

oc expose service hello  

oc get all  


<img width="822" height="373" alt="image" src="https://github.com/user-attachments/assets/97caca60-5064-4de9-84e0-ad10640d15ec" />



oc get pod -o wide  

oc scale --replicas 4 deployment/hello  

oc delete project chapter1



