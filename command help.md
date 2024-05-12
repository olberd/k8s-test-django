minikube start --driver=hyperv

minikube docker-env | Invoke-Expression

docker build -t django-k8s .

kubectl run django1 --image=django1 --image-pull-policy=Never

kubectl exec -ti django1 -- bash

docker inspect 95f6aa

psql -h localhost -p 5430 -U test_k8s -d test_k8s

ip к docker
172.18.112.1
psql -h 192.168.1.34 -p 5432 -U test_k8s -d test_k8s

docker
"IPAddress": "172.18.0.2",

192.168.1.34 ip моего компа

minikube ip

export DATABASE_URL=postgres://test_k8s:OwOtBep9Frut@192.168.1.34:5430/test_k8s

kubectl port-forward django1 7777:80

kubectl run django2 --image=django --env="DATABASE_URL=postgres://test_k8s:OwOtBep9Frut@192.168.1.34:5430/t
est_k8s"  --image-pull-policy=Never

kubectl run django-k8s --image=django-k8s --env="DATABASE_URL=postgres://test_k8s:OwOtBep9Frut@192.168.1.34:5430/t
est_k8s"  --image-pull-policy=Never

адрес сервиса
minikube service django-admin-service --url

или
minikube ip и порт

kubectl apply -f .\kubernetes\django-app.yaml

          env:
            - name: DEBUG
              value: "False"
            - name: SECRET_KEY
              value: adfasijvlkjvpoivzkjhfsda
            - name: DATABASE_URL
              value: postgres://test_k8s:OwOtBep9Frut@192.168.1.34:5430/test_k8s

kubectl run django-clearsessions --image=olberd/django-hub --restart=Never --dry-run=client -o yaml --command -- python
manage.py clearsessions
kubectl run django-clearsessions --image=olberd/django-hub --dry-run=client -o yaml --restart=Never --command python
manage.py clearsessions
kubectl run django-clearsessions --image=olberd/django-hub --command python manage.py clearsessions

kubectl run clearsessions --image=olberd/django-hub --env="DATABASE_URL=postgres://test_k8s:OwOtBep9Frut@192.168.1.34:
5430/t
est_k8s" --command -- ['python', 'manage.py', 'clearsessions']

kubectl run clearsessions --image=olberd/django-hub --command -- ['python', 'manage.py', 'clearsessions']
kubectl run clearsessions1 --image=olberd/django-hub
--command -- ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]

helm install dev-pg bitnami/postgresql --set global.postgresql.auth.postgresPassword=UterJes70 --set
global.postgresql.auth.username=test_k8s --set global.postgresql.auth.password=OwOtBep9Frut --set
global.postgresql.auth.database=test_k8s --set global.postgresql.service.ports.postgresql=5432

kubectl run dev-pg-postgresql-client --rm --tty -i --restart='Never' --namespace default --image
docker.io/bitnami/postgresql:16.3.0-debian-12-r4 --env="PGPASSWORD=OwOtBep9Frut" --command -- psql --host
dev-pg-postgresql -U test_k8s -d test_k8s -p 5432