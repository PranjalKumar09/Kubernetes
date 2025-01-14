this is chat application ok K8s -> 3 tier



react -> docker needed -> pod creation -> Deployement -> Service -> DBs
js - > docker needed ->  pod creation -> Deployement -> Service -> DBs
mongo -> docker needed  -> pod creation ->  Deployement -> Service -> DBs

these all will do in minikube


$  minikube start --driver=docker

this is application will do K8s -> https://github.com/iemafzalhassan/full-stack_chatApp

delting the k8s, .git, .gitignore folder/file, i will create fresh

 $ docker build -t pranjalkumar09/chatapp-backend:latest .

docker push pranjalkumar09/chatapp-backend:latest


 $ docker build -t pranjalkumar09/chatapp-frontend:latest .

docker push pranjalkumar09/chatapp-frontend:latest


and of mongo db we will use the mongo db official \\

 creating k8s folder in which create all k8 files

 cerating namespace 


$ kubectl create -f namespace.yml 

in deployment for containers (in spec) checking ports & env from docker files for each

and mongog db default port is 27017 and its own env like MONGO_INITDB_ROOT_USERNAME, MONGO_INITDB_ROOT_PASSWORD



in our project frontend is demanding nginx (backend) service 
{comments-> means this project above is tightly coupled , recommandtion..... }