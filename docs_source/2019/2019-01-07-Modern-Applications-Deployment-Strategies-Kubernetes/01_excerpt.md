[# Modern Applications Deployment Strategies with Kubernetes](/2019/2019-01-07-modern-applications-deployment-strategies-kubernetes/)

##### *7 January 2019*

[![Boulder Cascade Banner](/2019/assets/images/2019-01-07-Modern-Applications-Deployment-Strategies-Kubernetes/Boulder-Cascade-Banner.jpg)](/2019/2019-01-07-modern-applications-deployment-strategies-kubernetes/)

Applications deployment is coming at the end of the CI/CD pipeline. Nowadays, we have multiple tools available to help us deploy safely and with the less possible negative impact on our final users. Here, I am mainly thinking of Spinnaker, Terraform or Kubernetes, but there are many other tools.

However, it is still the most feared part of the complete application release process, and that's because we are possibly impacting our end users.

I had recently the chance to experiment different types of deployments to Kubernetes and would like to share them with you.
Here are the different deployment strategies available to you natively within Kubernetes today:

- rolling update
- recreate
- blue/green
