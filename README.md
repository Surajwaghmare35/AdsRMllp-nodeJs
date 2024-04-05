> [!IMPORTANT]  
> **Step 1** :
Docker need to installed to test setup locally by executing as:

```
docker-compose up -d
```
Then on browser type:
```
http://localhost:3000
```
> [!IMPORTANT]  
> **Step 2** : Here the docker image build locally in step 1, we need to push it in dockerHub registry as: \
You can push a new image to repository using the CLI:
```
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```
To test k8s hpa setup locally make sure you have k8s development cluster running,\
u can setup using minikube or kind. (I have setup using kind.)

**Now,** to make available k8s svc, deploy, po, hpa &nbsp; Execute:
```
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

