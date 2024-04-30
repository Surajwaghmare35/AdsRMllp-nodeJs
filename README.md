> [!IMPORTANT]  
> **Step 1** : Docker need to installed to test setup locally by executing as:

```
docker-compose up -d
```
Then on browser type: http://localhost:3000


> [!IMPORTANT]  
> **Step 2** : Here the docker image build locally in step 1, we need to push it in dockerHub registry as: \
> You can push a new image to repository using the CLI:

```
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```

To test k8s hpa setup locally make sure you have k8s development cluster running,\
u can setup using minikube or kind. (I have setup using kind.)

**Now,** to make available k8s svc, deploy, po, hpa &nbsp; Execute:

```
cd manifests
kubectl apply -f deployment.yaml,service.yaml,hpa.yaml,components.yaml
```

you will see output by executing below command as:

![image](https://github.com/Surajwaghmare35/adsremedyMediaLlp-NodeJs/assets/68895144/09413fb6-46ea-4dc5-a61f-005488679cd1)

Next, see how the autoscaler reacts to increased load

Run below command in a separate terminal\
so that the load generation continues and you can carry on with the rest of the steps

```
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://nodejs-svc:3000; done"
```

Then in another terminal, verify the result state (after a minute or so):

```
# type Ctrl+C to end the watch when you're ready
watch -x kubectl get hpa nodejs-hpa
```

if works well all, you will see complete steps as:

[k8s-hpa.webm](https://github.com/Surajwaghmare35/adsremedyMediaLlp-NodeJs/assets/68895144/e7ca5153-0bf3-4552-8347-7afc4fd57489)

> [!IMPORTANT]  
> **Step 3** : To deploy Postgresql, Grafana and Prometheus in k8s cluster follow steps as:

Note: **helm** need to install locally,
You can install it on os-specific by following Doc: https://helm.sh/docs/intro/install/

```
# lets apply k8s manifests
kubectl apply -f manifests

# you can check as:
watch -x kubectl get no,sc,pv,pvc,svc,deploy,po,ing,hpa,sts,cm,secrets -A
```

To add postgresql using helm follow :

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-postgresql bitnami/postgresql
watch -x kubectl logs pods/my-postgresql-0

# check pod status
watch -x kubectl get pods/my-postgresql-0

kubectl get secrets my-postgresql  -o jsonpath="{.data.postgres-password}" | base64 -d;echo
kubectl describe svc/my-postgresql | grep -inF Endpoints
helm list -a
```

Once pod in running state execute below: (optional)

```
kubectl run my-postgresql-client --rm --tty -i --restart='Never' \
--image docker.io/bitnami/postgresql:16.2.0-debian-12-r15 \
--env="PGPASSWORD=n0YUNS0Sx3" \
--command -- psql --host my-postgresql -U postgres -d postgres -p 5432

# replace PGPASSWORD value from my-postgresql secrets

# once inside psql shell check default db
\l # to list db's
\q # to exit
```

**After that**, install kube-prometheus-stack

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install my-kube-prometheus-stack prometheus-community/kube-prometheus-stack

watch -x kubectl get pods -l "release=my-kube-prometheus-stack"
```

Now, to install prometheus-postgres-exporter using helm as:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# create postgress-exporter-values.yaml as:

cat <<EOT | tee postgress-exporter-values.yaml
config:
  ## The datasource properties on config are passed through helm tpl function.
  ## ref: https://helm.sh/docs/developing_charts/\#using-the-tpl-function
  datasource:
    # Specify one of both datasource or datasourceSecret
    host: "10.244.0.8"
    user: postgres
    password: abSwwNOoxg

serviceMonitor:
  # When set true then use a ServiceMonitor to configure scraping
  enabled: true
  labels:
    release: my-kube-prometheus-stack
EOT

# note: Here replace val of pgsql endpoint-url ip,user,pass & helm release name.
```

do helm install

```
helm upgrade --install my-prometheus-postgres-exporter prometheus-community/prometheus-postgres-exporter -f postgress-exporter-values.yaml
kubectl get pods -l "app=prometheus-postgres-exporter,release=my-prometheus-postgres-exporter" -o jsonpath="{.items[0].metadata.name}";echo

watch -x kubectl get servicemonitors/my-prometheus-postgres-exporter -o yaml
# (in above highligtt helm/release)
```

(optional)

```
kubectl port-forward svc/my-prometheus-postgres-exporter --address 0.0.0.0 9187:80
kubectl describe svc/my-prometheus-postgres-exporter | grep -inF Endpoints
```

Now, to access ﻿grafana dahboard locally follow:

```
# note: default userName = admin
# to get grafana pass execute :
kubectl get secrets/my-kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d;echo

# after that, forward svc in seprate terminal
kubectl port-forward svc/my-kube-prometheus-stack-grafana --address 0.0.0.0 3000:80
```
On browser type: http://localhost:3000


Then, in grafana add dataSource --> choose --> postgres ---> add psql endpoint url ip,username,pass,db \
finally **Save&Test**.

Then in grafana for pg-exporter dashboard choose: 12485 or upload its json file

**Follow,** complete steps 3 steps including grafana as:

[k8s-helm-psql-prom-grafana.webm](https://github.com/Surajwaghmare35/adsremedyMediaLlp-NodeJs/assets/68895144/d3255cb4-36a9-428b-9b27-935c20d34978)

Note: helm uninstall not delete postgres pvs, to delete it manuallu do: kubectl delete pvc/data-my-postgresql-0

# DONE
