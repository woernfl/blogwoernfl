---
layout: post
title: How to provision persistent volume in Kubernetes.
feature-img: "assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/Storage-Banner.jpg"
thumbnail: "assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/Storage-Banner.jpg"
excerpt_separator: <!--more-->
tags: [Docker, Container, Kubernetes, Storage]
---

I recently got the chance to speak at DevoxxFR. I loved the event, and even more the chat I had in the hallway track. I went there to talk about the different ways of `managing stateful applications via Kubernetes`, this talk raised a lot of questions which inspired me to write this blog post.
<!--more-->

Most of this questions were about the `persistent volume` provisioning, so let's dive into this subject.

---

Basically there are 2 ways of managing it:

- `Manually creating the persistent volume`, which is leading to you creating all the persistent volume on your own (the last thing you want to do)
- `Automatically creating the persistent volume` by letting Kubernetes talk with your storage backend so that it can provision itself what it needs

If you do not know what is a `persistent volume`, that's ok <i class="fa fa-smile-o"></i>. Let say that you can see it as a network disk or a network drive, whatever rings a bell in your mind.

Some would argue that there is also the `statefulset` object and I would answer that from a state perspective the `statefulset` is using the automatic creation of the `persistent volume`, nothing more. Hoever `statefulset` are providing a number of feature usually useful when dealing with stateful applications. The most useful one, in my opinion, being the ordered deployment and deletion of pods.

---

# Let set some context

To make it easier to understand, let put some context here. Let's say that we are in a company called `BigCo`. `BigCo` has already rolled out `Kubernetes` to manage all services it provides. `John` and `Bob` are 2 employees of `BigCo`. `John` is a sysadmin and `Bob` is a developer. One day `Bob` come to `John` with an ask, he would need to run a MySQL database in the `Kubernetes` cluster hosting his application.

# Manually creating the persistent volumes

After some research `John` is ready to share a small POC with `Bob`. `John` just want to validate the concept, therefore he choose to create the persistent manually himself and don't get into the automatic persistent volume creation.

## The flow

So here is how the provisioning process work in this situation:

1.`BigCo` has a `Kubernetes` cluster and a storage backend up and running. They can both communicate together:

![PV+PVC-18]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/PV+PVC-18.jpg)
2.`John` create a `persistent volume`:

![PV+PVC-19]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/PV+PVC-19.jpg)
3.`Bob` create a `persistent volume claim` referencing the `persistent volume` previously created:

![PV+PVC-20]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/PV+PVC-20.jpg)
4.`Bob` create a `pod` and reference the `persistent volume claim` in this `pod`:

![PV+PVC-21]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/PV+PVC-21.jpg)
5.`Kubernetes` mount the `persistent volume` in the `pod`:

![PV+PVC-22]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/PV+PVC-22.jpg)

## The code

Tested on Minikube, if you are not running Minikube, there will some minor change to do, mainly in the `persistent volume` description.

1.Deploy a `namespace` (that's always the first thing to do):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mysql
```
2.Deploy a `persistent volume`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /data/mysql-pv/
```
3.Deploy a `persistent volume claim`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: mysql
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
4.Deploy a MySQL `pod`. Here we will be deploying it using a `deployment` object:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
        livenessProbe:
          exec:
            command: ["mysqladmin", "ping"]
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command: ["mysqladmin", "ping"]
          initialDelaySeconds: 5
          periodSeconds: 2
          timeoutSeconds: 1
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```
5.Expose internally to the cluster and via DNS the MySQL container via a `service`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
```
6.Test the MySQL database by connecting to it from inside the `Kubernetes` cluster using this command:
```sh
kubectl --namespace=mysql run -it --rm --image=mysql:5.7 --restart=Never mysql-client -- mysql -h mysql -ppassword
```

## The result

Now if the pod dies and is rescheduled somewhere else, the data will not be lost.

## Limitation

`John` had to create a persistent volume before `Bob` could do his job, in `BigCo`, that is usually meaning the `Bob` had to submit a ticket to `John`. Which is usually source of slowdown in the process, which we want to avoid.

---

# Automatically creating the persistent volumes

As `Bob` is happy with the solution proposed by `John`, he wants spin up more instances. `John` quickly need to industrialize the process and after some additional research, he put in place the automatique creation of the `persistent volume`.

## The flow

So here is how the provisioning process work in this situation: 

1.Again, `BigCo` has a `Kubernetes` cluster and a storage backend up and running. They can both communicate together:

![DP-24]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/DP-24.jpg)
2.`John` create one or many `storage class`, you can see `storage class` as type of storage (SSD, HDD, ...):

![DP-25]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/DP-25.jpg)
3.`Bob` create a `persistent volume claim` referring this time to one of the `storage class`:

![DP-26]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/DP-26.jpg)
4.`Kubernetes` create the `persistent volume` based on the `persistent volume claim` and the `storage class` chosen:

![DP-27]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/DP-27.jpg)
5.`Bob` create a `pod` and reference the `persistent volume claim` in the `pod`:

![DP-28]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/DP-28.jpg)
6.`Kubernetes` mount the `persistent volume` in the `pod`:

![DP-29]({{ site.baseurl }}/assets/img/posts/2018-09-20-How-to-provision-persistent-volume-in-Kubernetes/DP-29.jpg)

## The code

Here again tested on Minikube. Should also work on other `Kubernetes` cluster with minor changes to the `storage class`.

1.Deploy 2 `storage class`:
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  namespace: mysql
  name: hdd
provisioner: k8s.io/minikube-hostpath
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  namespace: mysql
  name: ssd
provisioner: k8s.io/minikube-hostpath
```
2.Deploy a `namespace`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mysql
```
3.Deploy a `persistent volume claim`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: mysql
spec:
  storageClassName: ssd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```
4.Deploy a MySQL `pod`, again using a `deployment` object: 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
        livenessProbe:
          exec:
            command: ["mysqladmin", "ping"]
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command: ["mysqladmin", "ping"]
          initialDelaySeconds: 5
          periodSeconds: 2
          timeoutSeconds: 1
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```
5.Expose the MySQL container via a `service`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
```
6.Test the MySQL database by connecting to it from inside the `Kubernetes` cluster using this command:
```sh
kubectl --namespace=mysql run -it --rm --image=mysql:5.7 --restart=Never mysql-client -- mysql -h mysql -ppassword
```

## The result

Same thing as previously, if the pod dies and is rescheduled somewhere else, the data will not be lost.

We even reused a lot of the code, basically, the biggest changes are the `storage class` introduction and the fact that the `persistent volume claim` is now referencing to the `storage class` instead of the `persistent volume`.

Additionally, once the many `storage class` have been created by `John`, `Bob` can create without delay the `persistent volume` he need.

## Limitation

As `persistent volume` are dynamically created, some people tend to forget to manage the all lifecycle, which can lead to `persistent volume` not deleted even if they are not used anymore. However, this can be easily worked around by reviewing from time to time the list of `persistent volume` instantiated and not linked to a `pod`.

---

# Conclusion

To sum up everything, use the automated way, as always. And if you want to manage stateful app on top of Kubernetes, Statefulset is worth a look.