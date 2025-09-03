# How to Deploy Application on OpenShift via CLI 

## Commands --  


oc new-app --name my-nginx registry.redhat.io/nginxinc/nginx-unprivileged 

# Deploy the Node.js image with a custom command
oc new-app --name my-nodejs-app --docker-image docker.io/library/node:18 \
  --entrypoint /bin/bash \
  -- sh -c 'echo "Hello from OpenShift!" > /tmp/index.html && http-server /tmp'

# Expose the application to the internet
oc expose service my-nodejs-app --port 8080

# Deploy the PostgreSQL image with a name and set environment variables.
oc new-app --name my-postgres --docker-image docker.io/library/postgres:15 \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb

oc new-project chapter1  

podman pull quay.io/redhattraining/todo-angular

docker pull quay.io/redhattraining/todo-angular

oc new-app --name hello1 quay.io/redhattraining/todo-angular

oc new-app --name hello --image quay.io/redhattraining/hello-world-nginx:v1.0  


<img width="832" height="357" alt="image" src="https://github.com/user-attachments/assets/b4bdde73-9e90-4edc-82a8-d2aaaa19c748" />

oc get all  

<img width="837" height="290" alt="image" src="https://github.com/user-attachments/assets/67291901-6f0c-41bf-aff0-3f6fcebfb1d1" />

oc expose service hello  

oc get all  


<img width="822" height="373" alt="image" src="https://github.com/user-attachments/assets/97caca60-5064-4de9-84e0-ad10640d15ec" />


<img width="852" height="440" alt="image" src="https://github.com/user-attachments/assets/8fcd06d4-d9b3-4f56-a8cf-98ae17d73e3b" />




oc get pod -o wide  

<img width="828" height="387" alt="image" src="https://github.com/user-attachments/assets/73528c13-602c-4b91-93a6-b2b5961bc110" />

<img width="822" height="475" alt="image" src="https://github.com/user-attachments/assets/15b08c81-d49a-459d-be0f-a5d0c20d6c86" />



oc scale --replicas 4 deployment/hello  

<img width="827" height="455" alt="image" src="https://github.com/user-attachments/assets/2d269218-0f66-4ec1-bc0d-c9cd7f723b35" />





oc delete project chapter1

<img width="822" height="296" alt="image" src="https://github.com/user-attachments/assets/5c809bc4-9093-452b-b974-d9fc38b7eb01" />




