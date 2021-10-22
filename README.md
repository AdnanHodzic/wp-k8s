# wp-k8s: WordPress on Kubernetes project

consists of WordPress Kubernetes deployments setup (MySQL cluster setup using MySQL operator, NFS, HPA, VPA, Ingress, cert-manager & Let’s Encrypt) which is made to be resilient and scalable, utilizing automation without need for maintenance. In situation of load increase, more pods and nodes would be added to the cluster, After the load is back to optimal levels, additional pods and nodes will be removed. 

While wp-k8s project is compatible with any Kubernetes/cloud provider, if you're interested in its GKE implementation, for more info refer to [wp-k8s: WordPress on Kubernetes (GKE, cloud SQL, NFS, cluster autoscaling, HPA, VPA, Ingress, Let's Encrypt)](http://foolcontrol.org/?p=3754) blog post.

## Brief overview:

`kustomization.yaml` - which will create all necessary secrets and make necessary deployments for following components:

* [nfs-server-k8s](https://github.com/AdnanHodzic/nfs-server-k8s) (`nfs.yaml`) which will store data to our Persistent storage
* MySQL Cluster deployment (`mysql-cluster.yaml`) using MySQL operator
* WordPress deployment (`wordpress-deployment.yaml`) which connects to external MySQL database for data
* VPA (`vpa.yaml`) - enables Vertical Pod Autoscaler inUpdateMode: Offmode, which instead vertically scaling our deployments will give recommendations on type of resources are deployments are using. Which will help us set more realisticresourcestype. Which will help HPA/metrics server make better auto scaling calculations and decision.
* HPA (`hpa.yaml`) - enables Horizontal Pod Autoscaler, which will scale our pods to desired number to reduce/balance the load. In case our pods need more system resources to deploy new pods, cluster-autoscaler will step in and provision and new nodes to our cluster.
* ingress (`ingress.yaml`)with ingress-nginx controller which will serve as our load balancer. This will be our only entry point to our cluster from outside world on port 443 (any traffic on port 80 will be redirected to 443).
* cert-manager which will automatically issue and renew Let's Encrypt certificate to have secure HTTPS access to our WordPress deployment

## Deploying wp-k8s to your Kubernetes cluster

#### Prerequisites: 
* Kubernetes cluster, kubectl and helm cli tools
* A sub/domain name with access to edit its DNS records
* Contents of this repo (`git clone https://github.com/AdnanHodzic/wp-k8s.git`)

### Step 1: MySQL database server

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

### Step 2: WordPress deployment

Fill out kustomization.yaml file to reflect values which were made as part of "Step 1" so WordPress can connect to MySQL cluster and run deployment:

`kubectl apply -k ./`

#### What was just deployed? 

* nfs.yaml – will create NFS server using nfs-server-k8s, for further reference refer to [“Solution (to all 3 problems)” as part of "Step 3: NFS & Persistent volume/disk" section.](https://foolcontrol.org/?p=3754)  
* wordpress-deployment.yaml – will create WordPress deployment using (official WordPress docker image) and its service. It will pick up secrets & variables set as part of `kustomization.yaml` file. As part of a ConfigMap, various changes which to allow upload of size up 64MB. Which will fix problems of not being able to upload large files due to their file size or image dimension.
* vpa.yaml – will create VPA whose function has been described in "Brief overview" section.
* hpa.yaml – will create HPA whose function has been described in "Brief overview" section
* ingress.yaml – will create an Ingress whose functionality will be describe in next step "Step 3: Ingress"

### Step 3: Ingress

#### Install nginx-ingress
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress ingress-nginx/ingress-nginx \
--set 'ingress.annotations.nginx\.ingress\.kubernetes\.io/client-max-body-size=40m'
```

#### Update DNS records for target sub/domain name

Watch status and wait to get ingress controller's exposed external IP:
`kubectl get services -w ingress-ingress-nginx-controller`

Copy exposed external IP and add/update DNS A record for your domain, i.e:

![Ingress external IP](https://foolcontrol.org/wp-content/uploads/2021/10/ingress-external-ip.png)

At this point your site will be accessible, but I still don’t encourage to enter any details/proceed with WordPress installation until we have secure HTTPS connection described in next step.

### Step 4: cert-manager & Let’s Encrypt

In this step we'll configure [cert-manager](https://cert-manager.io/) to request and automatically renew Let's Encrypt certificate for domain used in previous step.

#### Create cert-manager namespace
`kubectl create namespace cert-manager`

#### Install cert-manager
```
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set installCRDs=true
```

#### Verify installation
By making sure all cert-manager* workloads are up and running:
`watch kubectl get pods -n cert-manager`

#### Create/issue certificates

##### Create Let's Encrypt certificate issuer `staging` (optional)

`kubectl apply -f cluster-issuer-staging.yaml`

While this step is optional, it's recommended to first create staging certificate which will create self-signed (insecure) certificate using [Let's Encrypt certificate staging environment](https://letsencrypt.org/docs/staging-environment/). This is solely used to test if everything is setup correctly with DNS and ingress, before proceeding withproductionand avoid reaching [Let's Encrypt Rate Limits.](https://letsencrypt.org/docs/rate-limits/)

##### Create Let's Encrypt certificate issuer `production`

`kubectl apply -f cluster-issuer.yaml`

##### Check certificate request was successful

`kubectl describe certificate wordpress-tls`

##### Wait until your Let's Encrypt certificate is ready

`watch kubectl get certificate`

Once `Ready` column becomes `True` (this can take few minutes) domain name you set "pdate DNS records for target sub/domain name" will be available via HTTPS using Let's Encrypt TLS certificate. With this done, your wp-k8s (WordPress on Kubernetes) has been complete!

![wp-k8s deployment is complete!](https://foolcontrol.org/wp-content/uploads/2021/10/valid-lets-Ecrypt-certificate.png)


## Discussion:

* Blogpost: [wp-k8s - WordPress on Kubernetes (GKE, cloud SQL, NFS, cluster autoscaling, HPA, VPA, Ingress, Let's Encrypt)](http://foolcontrol.org/?p=3754)

## Donate

If you found this project useful, show your support and appreciation by donating or contributing code. Otherwise, giving credits and acknowledgments also goes a long way.

### Financial donation

If wp-k8s helped you out and you find it useful, show your appreciation by donating (any amount) to the project!

##### PayPal
[![paypal](https://www.paypalobjects.com/en_US/NL/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/donate?business=7AHCP5PU95S4Y&no_recurring=0&item_name=Purpose%3A+Contribution+for+work+on+wp-k8s&currency_code=EUR)

##### BitCoin
[bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87](bitcoin:bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87)

[![bitcoin](https://foolcontrol.org/wp-content/uploads/2019/08/btc-donate-displaylink-debian.png)](bitcoin:bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87)

### Code contribution

Other ways of supporting the project consists of making a code or documentation contribution. If you have an idea for a new features or want to implement some of the existing feature requests or fix some of the [bugs & issues](https://github.com/AdnanHodzic/wp-k8s/issues). Please make your changes and submit a [pull request](https://github.com/AdnanHodzic/wp-k8s/pulls) which I'll be glad to review. If your changes are accepted you'll be credited for your contiribution.
