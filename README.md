# wp-k8s: WordPress on Kubernetes project

This repository consists of various Kubernetes deployments (see below for full info) which aim to seamlessly deploy production ready WordPress on both public and private cloud with autoscaling capabilities with minimum to no ops tasks. If you're interested in same type of setup without K8s, please refer to my [containerized-wordpress-project](https://github.com/AdnanHodzic/containerized-wordpress-project).

While wp-k8s project is compatible with any Kubernetes/cloud provider:

#### For **wp-k8s private cloud** implementation refer to: 

* [wp-k8s: WordPress on privately hosted Kubernetes cluster (Raspberry Pi 4 + Synology)](https://foolcontrol.org/?p=4004) or
* [rpi-microk8s-bootstrap: Automate RPI device conversion into Kubernetes cluster nodes with Terraform](https://foolcontrol.org/?p=4555) blog post
* [rpi-microk8s-bootstrap](https://github.com/AdnanHodzic/rpi-microk8s-bootstrap) Github project

#### For **wp-k8s public cloud** implementation on GKE refer to:

* [wp-k8s: WordPress on Kubernetes (GKE, cloud SQL, NFS, cluster autoscaling, HPA, VPA, Ingress, Let's Encrypt)](http://foolcontrol.org/?p=3754) blog post

#### For running **serverless WordPress with Cloud Run** refer to 

* [wp-cloud-run: Ultimate WordPress setup on (GCP) Cloud Run](https://foolcontrol.org/?p=4802) blog post or its:
* [wp-cloud-run 14 video Youtube playlist](https://www.youtube.com/playlist?list=PL83G0TLSeXREwjHDZPsV_34azAmniL81V).

## Brief overview:

By Default, [kustomization.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/kustomization.yaml) - will create all necessary secrets and make necessary deployments. See its contents for which deployments to enable based on which cloud (private or public) is targeted.

Contents of this file can be deployed by running: `kubectl apply -k ./`

## Full overview

* [metallb-ingress-service.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/metallb-ingress-service.yaml) – explained as part of [Step 4.8](https://foolcontrol.org/?p=4004)
* [mysql-cluster.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/mysql-cluster.yaml) - to be used only if you’re not using Synology MariaDB or Cloud SQL database and want to create MySQL cluster as part of k8s cluster. See [HowTo Create MySQL Cluster using mysql-operator](https://github.com/AdnanHodzic/wp-k8s#howto-create-mysql-cluster-using-mysql-operator) section for more info.
* [nfs.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/nfs.yaml) - will utilize [NFS server container image for Kubernetes](https://github.com/AdnanHodzic/nfs-server-k8s), explained as part of [Problem 3.3](https://foolcontrol.org/?p=3754).
* [nfs-synology.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/nfs-synology.yaml) – Explained as part of [Step 5: Configure NFS on Synology](https://foolcontrol.org/?p=4004)
* [wordpress-deployment.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/wordpress-deployment.yaml) – will create WordPress deployment using (official WordPress docker image) and its service. It will pick up secrets & variables set as part of “kustomization.yaml” file. Deployment will utilize our previously created NFS PVC. As part of a ConfigMap, various changes which to allow upload of size up 64MB. Which will fix problems of not being able to upload large files due to their file size or image dimension.
* [ingress.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/ingress.yaml) with ingress-nginx controller which will serve as our load balancer. This will be our only entry point to our cluster from outside world on port 443 (any traffic on port 80 will be redirected to 443).
* [cluster-issuer.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/cluster-issuer.yaml) Explained as part of [Step 6.5](https://foolcontrol.org/?p=3754)
* [cluster-issuer-staging.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/cluster-issuer-staging.yaml) Explaiend as part of [Step 6.5](https://foolcontrol.org/?p=3754)
* [vpa.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/vpa.yaml) – will create VPA whose function has been described in [wp-k8s: WordPress on Kubernetes project)](https://foolcontrol.org/?p=3754) section.
* [hpa.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/hpa.yaml) – will create HPA whose function has been described in [wp-k8s: WordPress on Kubernetes project](https://foolcontrol.org/?p=3754)
* [stateless-sa-sec.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/stateless-sa-sec.yaml) - Will create Kubernetes secret named `wp-stateless-media-secret` for use as part of [WP-Stateless (Google Cloud Storage) WordPress plugin](https://github.com/udx/wp-stateless). Make sure changes are made accordingly to [wordpress-deployment.yaml](https://github.com/AdnanHodzic/wp-k8s/blob/main/wordpress-deployment.yaml) file where `stateless-sa-sec.yaml` is referenced. Referenced as part of [Step 3.1](https://foolcontrol.org/?p=3754) & [Step 5](https://foolcontrol.org/?p=4004)

## HowTo Create MySQL Cluster using mysql-operator

If you don't have an existing MySQL database server, you can create MySQL cluster using [MySQL operator](https://github.com/bitpoke/mysql-operator).

#### Install MySQL Operator
```
helm repo add bitpoke https://helm-charts.bitpoke.io
helm install mysql-operator bitpoke/mysql-operator
```

#### Deploy MySQL cluster

Replace every "redacted" value as part of mysql-cluster.yaml with base64 encoded value, i.e:
```
echo -n 'mysql-password-value' | base64
YlhsemNXd3RjR0Z6YzNkdmNtUT0K
```

Followed by: `kubectl apply -f mysql-cluster.yaml`

After couple of minutes your MySQL cluster will be up and running. For more informaiton refer to: [MySQL Operator - Deploy Cluster page](https://github.com/bitpoke/mysql-operator/blob/master/docs/deploy-mysql-cluster.md)

## Discussion:

* [wp-k8s: WordPress on privately hosted Kubernetes cluster (Raspberry Pi 4 + Synology)](https://foolcontrol.org/?p=4004) blog post
* [wp-k8s - WordPress on Kubernetes (GKE, cloud SQL, NFS, cluster autoscaling, HPA, VPA, Ingress, Let's Encrypt)](http://foolcontrol.org/?p=3754) blog post
* [rpi-microk8s-bootstrap: Automate RPI device conversion into Kubernetes cluster nodes with Terraform](https://foolcontrol.org/?p=4555) blog post


## Donate

If you found this project useful, show your support and appreciation by donating or contributing code. Otherwise, giving credits and acknowledgments also goes a long way.

### Financial donation

If wp-k8s helped you out and you find it useful, show your appreciation by donating (any amount) to the project!

##### Become Github Sponsor

[Become a sponsor to Adnan Hodzic on Github](https://github.com/sponsors/AdnanHodzic) to acknowledge my efforts and help project's further open source development.

##### PayPal
[![paypal](https://www.paypalobjects.com/en_US/NL/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/donate?business=7AHCP5PU95S4Y&no_recurring=0&item_name=Purpose%3A+Contribution+for+work+on+wp-k8s&currency_code=EUR)

##### BitCoin
[bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87](bitcoin:bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87)

[![bitcoin](https://foolcontrol.org/wp-content/uploads/2019/08/btc-donate-displaylink-debian.png)](bitcoin:bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87)

### Code contribution

Other ways of supporting the project consists of making a code or documentation contribution. If you have an idea for a new features or want to implement some of the existing feature requests or fix some of the [bugs & issues](https://github.com/AdnanHodzic/wp-k8s/issues). Please make your changes and submit a [pull request](https://github.com/AdnanHodzic/wp-k8s/pulls) which I'll be glad to review. If your changes are accepted you'll be credited for your contiribution.
