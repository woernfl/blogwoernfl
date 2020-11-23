---
layout: post
title: Modern Applications Deployment Strategies with Kubernetes
feature-img: "assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/boulder-cascade-banner.jpg"
thumbnail: "assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/boulder-cascade-banner.jpg"
excerpt_separator: <!--more-->
tags: [Application Deployment, Kubernetes]
---

Applications deployment is coming at the end of the CI/CD pipeline. Nowadays, we have multiple tools available to help us deploy safely and with the less possible negative impact on our final users. Here, I am mainly thinking of Spinnaker, Terraform or Kubernetes, but there are many other tools.
<!--more-->

However, it is still the most feared part of the complete application release process, and that's because we are possibly impacting our end users.

I had recently the chance to experiment different types of deployments to Kubernetes and would like to share them with you.
Here are the different deployment strategies available to you natively within Kubernetes today:

- rolling update
- recreate
- blue/green

---

# Rolling Update

This strategy is the one used by default in Kubernetes. It consists in creating a new replicat of the application before removing an old one and repeat this operation as many time as needed.

## The flow

1. We have a `V1` of an application already deployed on our cluster and served by a service: ![RU-1]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RU-1.png)
2. We apply an updated deployment definition, which create a `V2` deployment and a first `V2` pod, once this `V2` first pod is healthy, Kubernetes begins to serve traffic: ![RU-2]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RU-2.png)
3. At this stage one of the `V1` pods is not needed anymore and is removed: ![RU-3]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RU-3.png)
4. Kubernetes continues the rollout of the new version of our application by provisioning a new `V2` instance: ![RU-4]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RU-4.png)
5. At the end we rollout all the pods of the `V2` version of the app and have deleted all the pods of the `V1` version: ![RU-5]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RU-5.png)

> ### Pros
> - Easy to put in place
> - No downtime
> ### Cons
> - Slow deployments

## The code

The deployment strategy has to be specified in the deployment manifest.

For the rolling update strategy we can tune 2 settings:

- `maxSurge`: define how many pods we can add to the desired replica number at an instant *t*
- `maxUnavailable`: define how many pods can be unavailable at an instant *t*

In large deployment with spare cpu and ram capacities, you can bump up the `maxSurge` number to speed up the new version release.

Here is what would such a deployment look like:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  ##### DEPLOYMENT STRATEGY START #####
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  ##### DEPLOYMENT STRATEGY END #####
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: registry.gitlab.com/woernfl/simple-web-app:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        resources:
          requests:
            memory: "50Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
        livenessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
```

---

# Recreate

This strategy consists in creating new replica of the application after removing the old ones. In this case the additional resource consumption needed during the update is null and the operation can be fairly quick. However, updating using the `Recreate` strategy causes a downtime.

I found this type of update particularly useful if you need one pod to have access to one persistent volume at a time. You can find an example of this usage [here](https://github.com/woernfl/k8s-stateful-demo){:target="_blank"}

## The flow

1. Again we have a `V1` of an application already deployed on our cluster and served by a service: ![RE-1]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RE-1.png)
2. We apply the `V2` version, which triggers the deletion of the `V1` deployment and pods: ![RE-2]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RE-2.png)
3. The `V2` pods are created and served once healthy: ![RE-3]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/RE-3.png)

> ### Pros
> - Easy to put in place
> - Fairly quick deployment
> ### Cons
> - Downtime

## The code

The only manifest we need to modify here is the deployment one and there is not that much that we can customise.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  ##### DEPLOYMENT STRATEGY START #####
  strategy:
    type: Recreate
  ##### DEPLOYMENT STRATEGY END #####
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: registry.gitlab.com/woernfl/simple-web-app:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        resources:
          requests:
            memory: "50Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
        livenessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
```

---

# Blue/Green

The deployment strategy consists into first creating all the components of a `V2` application before switching traffic to it and deleting the `V1` application.

## The flow

1. As always, we have the same starting point, a `V1` of an application already deployed on our cluster and served by a service: ![BG-1]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/BG-1.png)
2. We begin by creating a new deployment of the `V2` application: ![BG-2]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/BG-2.png)
3. Once the `V2` deployment is complete and healthy, we update our service to serve the `V2` application: ![BG-3]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/BG-3.png)
4. Once we do not need the `V1` deployment anymore (eg: we are sure we do not need to rollback), we delete the `V1` deployment: ![BG-4]({{ site.baseurl }}/assets/images/posts/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/BG-4.png)

> ### Pros
> - Quick roll back available
> - No versioning/state issue
> ### Cons
> - High resource consumption
> - Not native to Kubernetes (more steps to orchestrate manually)

## The code

We will need to have 3 manifests in this case, one for the `V1` deployment, a second one for the `V2` deployment and a last one for the service used to expose the deployments.

Here is the `V1` deployment to get the initial deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ##### THE DEPLOYMENT NAME NEEDS TO BE UNIQUE #####
  name: web-app-deployment-v1
  ##### THE DEPLOYMENT NAME NEEDS TO BE UNIQUE #####
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
      ##### WE SPECIFY A UNIQUE POD SELECTOR FOR THE VERSION #####
      version: V1
      ##### WE SPECIFY A UNIQUE POD SELECTOR FOR THE VERSION #####
  template:
    metadata:
      labels:
        app: web-app
        ##### WE SPECIFY A UNIQUE POD LABEL FOR THE VERSION #####
        version: V1
        ##### WE SPECIFY A UNIQUE POD LABEL FOR THE VERSION #####
    spec:
      containers:
      - name: web-app
        image: registry.gitlab.com/woernfl/simple-web-app:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        resources:
          requests:
            memory: "50Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
        livenessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
```

Our service manifest to expose our `V1` deployment:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  labels:
    app: web-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: web-app
    ##### WE SPECIFY A THE UNIQUE POD LABEL THE SERVICE NEED TO TARGET #####
    version: V1
    ##### WE SPECIFY A THE UNIQUE POD LABEL THE SERVICE NEED TO TARGET #####
```

When we want to update our app version, here is our `V2` deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ##### THE DEPLOYMENT NAME NEEDS TO BE UNIQUE #####
  name: web-app-deployment-v2
  ##### THE DEPLOYMENT NAME NEEDS TO BE UNIQUE #####
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
      ##### WE SPECIFY A UNIQUE POD SELECTOR FOR THE VERSION #####
      version: V2
      ##### WE SPECIFY A UNIQUE POD SELECTOR FOR THE VERSION #####
  template:
    metadata:
      labels:
        app: web-app
        ##### WE SPECIFY A UNIQUE POD LABEL FOR THE VERSION #####
        version: V2
        ##### WE SPECIFY A UNIQUE POD LABEL FOR THE VERSION #####
    spec:
      containers:
      - name: web-app
        image: registry.gitlab.com/woernfl/simple-web-app:2.1
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        resources:
          requests:
            memory: "50Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
        livenessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 20
          timeoutSeconds: 1
```

Once our `V2` deployment is healthy and ready to serve traffic, we update our service manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  labels:
    app: web-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: web-app
    ##### WE SPECIFY THE NEW UNIQUE POD LABEL THE SERVICE NEED TO TARGET #####
    version: V2
    ##### WE SPECIFY THE NEW UNIQUE POD LABEL THE SERVICE NEED TO TARGET #####
```

Don't forget to delete the `V1` deployment once you are sure it is not needed anymore <i class="fa fa-smile-o"></i>

---

# Conclusion

Choose the best strategy for your use case. The `rolling update` one will suits 90% of them, the `recreate` one will be particularly useful if you are managing state and a downtime is accessible, the `blue/green` one is certainly the safest.

If you want to have more control over your deployments, there are much more update strategies options that are available if you are using service mesh, the `canary release` one or the `A/B testing` one are some of the interesting ones.
