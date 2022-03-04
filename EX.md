# EX280 ver 4.6 V3

---

:::info
Thanks for Nelson and Catherine's effort
:::

:::success
**IMPORTANT**
* For those who want to participle this exam have to familiar w/ `Deployment Config`
* Vim Editor, type `set paste:` to correct format before edit
* *Good Practice* - Each question has a specific namespace, please make sure you are doing the right thing in the correct namespace.
:::


### 0. Configure Your Environment
* Some Important Parameters 
    * `workbench domain name`
    * `root` and `ocpadm` `password`
    * `cluster API name`
    * `kubeadmin password` , `/root/kubeadmin-passwd`
* Docmentations
    * AU 6.1.2 [Install httpd-tools and Configuring identity providers](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#identity-provider-creating-htpasswd-file-linux_configuring-htpasswd-identity-provider)
    * AU 6.1.5 [oauth/cluster sample YAML](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#identity-provider-htpasswd-CR_configuring-htpasswd-identity-provider)
    * AU 5.3 [Remove Kubeadmin user](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#removing-kubeadmin_understanding-identity-provider)
    * AP 4.1.5 [Resource Quota](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/applications/index#quotas-sample-resource-quota-definitions_quotas-setting-per-project)
    * NO 7.3 [LimintRanges](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/nodes/index#nodes-cluster-limit-ranges)
  
```shell=
## Login w/ root to fetch kube
$ ssh root@workbench.xx.xx.com
passwd: pflxxxg1
$ cat /root/kubeadmin-passwd
kubeadmin-passwd
```
--- 
### 1. Config your HTPasswrd identity provider `ex280-htpasswd`, using the secret `ex280-secret`, and create following users w/ password, 
- [ ] `armstrong` w/ password `p1`
- [ ] `jobs` w/ password `p2`
- [ ] `worxxx` w/ password `p3`
- [ ] `aldrin` w/ password `p4`
- [ ] `u5` w/ password `p5`
- [ ] `u6` w/ password `p6`

* [Install httpd-tools](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#identity-provider-creating-htpasswd-file-linux_configuring-htpasswd-identity-provider) - 這題金夭壽，坑爆！！！！！我第一次考，前一小時沒有htpasswd可用，狂敲Linux指令，我以為我在考EX134
```shell=
## Login w/ ocpadm
$ ssh ocpadm@workbench.xx.xx.com
passwd: pflxxxg1

$ sudo yum install -y httpd-tools
```
* [Creating an HTPasswd file using Linux](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#identity-provider-creating-htpasswd-file-linux_configuring-htpasswd-identity-provider)

```shell=
## 1st User
$ htpasswd -c -B -b localusers armstrong p1

## Following users
$ htpasswd -B -b localusers jobs p2
....
....
```
* [Creating the HTPasswd Secret](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#identity-provider-creating-htpasswd-secret_configuring-htpasswd-identity-provider)
```shell=
## login w/ kubeadmin
$ oc login -u kubeadmin -p kubeadmin-passwd

## Create secret
$ oc create secret generic ex280-secret --from-file=htpasswd=localusers -n openshift-config
```

* Replace `OAuth/Cluster`, [Sample HTPasswd CR](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#identity-provider-htpasswd-CR_configuring-htpasswd-identity-provider) shown here.
```shell=
## Modify Oauth/Cluster
$ oc get oauth cluster -o yaml > oauth.yaml

## Edit auth.ayml
$ vi oauth.yaml
```
```yaml=
# oauth.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: ex280-htpasswd      ## identity provider name
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: ex280-secret    ## secret name
```

```shell=
$ oc apply -f oauth.yaml

## (Optional) Check pod
$ oc get po -n openshift-config -w
## 2 pods restart

## (Optional) Check each user can login w/ specfic password
$ oc login -u jobs -p xxx
```


### 2. Configure Cluster Role Policy
- [ ] User `jobs` is `cluster-admin`
- [ ] User `armstrong` can create project,
- [ ] User `armstrong` cannot manage cluster
- [ ] User `aldrin` cannot create project
- [ ] User `kube-admin` does not exit



* User `jobs` is `cluster-admin`
```shell=
$ oc adm policy add-cluster-user-to-user cluster-admin jobs
```

* `armstrong` can create project
```shell=
$ oc adm policy add-cluster-user-to-user self-provisioner armstrong
```

* User `aldrin` cannot create project
```shell=
## In the other word, remove `self-provisioner` from `system:authenticated:oauth` group
$ oc adm policy remove-cluster-role-from-group \
>    self-provisioner system:authenticated:oauth
```


* [Removing the kubeadmin user](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/authentication_and_authorization/index#removing-kubeadmin_understanding-identity-provider)
```shell=
## Login w/ new cluster admin jobs
$ oc login -u jobs -p jobs-password

## Remove kubeadmin secret
$ oc delete secrets kubeadmin -n kube-system
```

### 3. Configure Projects and accessibility
- [ ] Create 5 projects
    - [ ] Project `apollo` exist
    - [ ] Project `p2` exist
    - [ ] Project `p3` exist
    - [ ] Project `p4` exist
    - [ ] Project `p5` exist
- [ ] User `armstrong` is admin of `p1`
- [ ] User `woxx` can edit `p2`
- [ ] User `u3` and `u4` can only view `p3`
    

* Create Projects
```shell=
$ oc new-project apollo
$ oc new-project p2
$ oc new-project p3
$ oc new-project p4
$ oc new-project p5
```

* Create Policy
```shell=
## Admin
$ oc policy add-role-to-user admim armstrong -n apollo

## edit in p2 project
$ oc policy add-role-to-user edit woxx -n p2

## view in p3 project
$ oc policy add-role-to-user view u3 -n p3
$ oc policy add-role-to-user view u4 -n p3
```



### 4. Configure Group access
- [ ] Create 5 groups, `commander`, `pilot`, `g3`, `g4` and `g5`
- [ ] User `armstrong` in `commander`
- [ ] User `woxx` and `u3` in `pilot`
- [ ] Group `commander` can edit `apollo` project
- [ ] Group `pilot` can view `apollo` project



* Create groups
```shell=
$ oc adm groups new commander
$ oc adm groups new pilot
$ oc adm groups new g3
$ oc adm groups new g4
$ oc adm groups new g5
```
* Add user to group
```shell=
$ oc adm groups add-users commander armstrong
$ oc adm groups add-users pilot woxx
$ oc adm groups add-users pilot u3
```

* Create Policy
```shell=
## edit role in apollo project
$ oc policy add-role-to-user edit commander -n apollo

## view role in apollo project
$ oc policy add-role-to-user view pilot -n apollo
```


### 5. Configure Resource Quota
- [ ] Create ResourceQuota named `ex280-quota`at project`p3`
- [ ] Projects are limited to 3 pods
- [ ] Projects are limited to 3 services
- [ ] Projects are limited to 6 replicationcontrollers
- [ ] Projects can request a maximum of 100m and can use maximum of 2 full core CPUs.
- [ ] Projects can request a maximum of 1Gi memory and can use maximum of 2Gi memory
* [Sample resource quota](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/applications/index#quotas-sample-resource-quota-definitions_quotas-setting-per-project) shown as below

```shell=
## Switch to project p3
$ oc project p3



$ vi quota.yaml
```

```yaml=
# quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ex280-quota        ## quota name here
spec:
  hard:
    pods: "3"
    requests.cpu: "100m"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    replicationcontrollers: "6"
    services: "3"
```

```shell=
$ oc apply -f quota.yaml
```


### 6. Configure Limit Ranges
- [ ] Create a limit range named `ex280-lomits` at project `p4`
- [ ] Each container requests 30m to 300m of CPU, default request is 100m of CPU
- [ ] Each container requests 5Mi to 500Mi of memory, default request is 100Mi of memory 
- [ ] Each Pod requests 30m to 300m of CPU
- [ ] Each Pod requests 5Mi to 500Mi of memory

* [Sample limit ranges](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/nodes/index#nodes-cluster-limit-ranges)

```shell=
## Switch to project p4
$ oc project p4

## Create limits.yaml
$ vi limits.yaml
```

```yaml=
# limits.yaml
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "ex280-lomits"        # Limits name here
spec:
  limits:
    - type: "Container"
      max:
        cpu: "300m"
        memory: "500Mi"
      min:
        cpu: "30m"
        memory: "5Mi"
      defaultRequest:        # not sure is request or defulatRequest
        cpu: "100m"
        memory: "100Mi"
    - type: "Pod"
      max:
        cpu: "30m"
        memory: "5Mi"
      min:
        cpu: "300m"
        memory: "500Mi"
```

```shell=
$ oc apply -f limits.yaml
```

### 7. Deploy Application
At project `p7` deply and application `oxcart`
- [ ] The host name of application route is `oxcart-p7.apps.ocp4.example15.com`
- [ ] Application can produce output

* remove node taint
```shell=
## Switch to project p7
$ oc project p7

## I forgot the taint key, you can use oc edit node <work0.name> to find the taint key
$ oc adm taint nodes <work0.name> node-
$ oc adm taint nodes <work1.name> node-
```

* re-assign hostname
```shell=
$ oc delete route oxcart

## The service oxcart has 2 port exposed, select 8080 port
$ oc exposer svc/oxcart --host-name oxcart-p7.apps.ocp4.example15.com --port 8080
```


### 8. Manual scale `math` deployment to 5 pods in project `p8` 
```shell=
## Switch to project p7
$ oc project p8

## Scale to 5 replicas
$ oc scale dc/math --replicas=5
```


### 9. Create hpa for deployment `hydra` in `lerna` project
- [ ] The deployment must have at least 6 pods running. If the average CPU utilization exceeds60%, then the deployment scales to a maximum of 9 pods.
- [ ] requests 25m to 100m of CPU

* Edit deploymentConfig
```shell=
## Switch to project lerna
$ oc project lerna

## Edit deployment config
$ oc edit dc/hydra
```

```yaml=
...output omitted..
resources:
  requests:
    cpu: 25m
    memory: 100Mi        # although memory not mensioned, you have to specify it, or hpa won't work
  limits:
    cpu: 100m
    memory: 500Mi
...output omitted..
```


```shell=
## Create HPA
$ oc autoscale dc/{dc_name} --min 5 --max 9 --cpu-percent 60

## (Optional)
$ oc get hpa -w
## Until cpu have no <unknown> value
```

### 10. Secure your application `ardin` at project `area51`
- [ ] subject, `/C=US/ST=NV/L=Hico/O=CIA/CN=ardin.ocp4.example15.com`
- [ ] The application can be accessed at https://ardin.ocp4.example15.com

We provide a tool `xxxcert` to assit you to generate csr, key and crt


* Create cert
```shell=
## Switch to project area51
$ oc project area51

## Using xxxcert to generage key and cer
$ xxxcert /C=US/ST=NV/L=Hico/O=CIA/CN=ardin.ocp4.example15.com

$ ls
domain.csr
domain.key
domain.crt
```


* Create edge route
```shell=
$ oc create route edge --service hydra --hostname ardin.ocp4.example15.com \
>  --cert domain.crt \
>  --key domain.key -n area51 --port 8080
```

### 11. Configure Secret
- [ ] Create a secret named `s11` in project `gru`
- [ ] The secret has a key `my_key_11`
- [ ] The secret has a value `XZDWEAQW78QWRE=`

* Create secret
```shell=
## Swtich to project gru
$ oc project gru

$ oc create secret generic s11 --from-literal my_key_11=XZDWEAQW78QWRE=
```

### 12. Deploy Application
- [ ] This application `a12` need a secret MY_KEY_11
- [ ] From browser access this application, whould not show the string `This application does not configured properly`
```shell=
$ oc set env dc/a12 --from secret/s11
```

### 13. Configure  Service account
- [ ] Create Service account named `ex280-sa` in project `p13` that any user can run container (anyuid)

```shell=
## Switch to project p13
$ oc project p13

## Create service account
$ oc create sa ex280-sa

## Apply scc:anyuid
$ oc adm policy add-scc-to-user anyuid -z ex280-sa
```

### 14. Deploy Application
- [ ] Apply service account `ex280-sa` to application `oranges` in project `p13`
- [ ] Do not add or delete any cofig in the application
- [ ] The application can produce output


* Set Service account
```shell=
$ oc set sa dc/oranges ex280-sa
```

* Correct label selectors at service
```shell=
## Modify serive selector
$ oc edit svc/oragens
```

```yaml=
...output omitted...  
    selector:
      app: oranges        ## app=orange -> app=oranges

...output omitted...  
```


### 15. Deployment Application
- [ ] At project `voyager`, there is a application `a15`
- [ ] Do not add or delete any cofig in the application
- [ ] The application can produce output


* Correct node selector
```shell=
## Switch to project voyager
$ oc project voyager

## or to see correct answer
$ oc get nodes -L star

$ oc edit dc/a15
```

```yaml=
...output omitted...      
      dnsPolicy: ClusterFirst
      nodeSelector:
        star: trek                    # this line,  star=Trek` -> `star=trek`
      restartPolicy: Always
...output omitted...
```
* Correct Route
```shell=
## the route is app.example15.com, should be apps.example15.com
$ oc delete route a15

## Create new route instead
$ oc expose svc/a15 --host-name a15-voyager.example15.com --port 8080
```


### 16. Deploy Application
- [ ] At project `mercury` there is a application `a16`
- [ ] Do not add or delete any cofig in the application
- [ ] The application can produce output

```shell=
## Switch to project mercury
$ oc project mercury

## edit Deployment Config

$ oc edit dc/a15
```

from
```yaml=
...output omitted..
resources:
  requests:
    memory: 80Gi ## Modify this line
...output omitted..
```
to

```yaml=
...output omitted..
resources:
  requests:
    memory: 80Mi     ## 80 Mi is okay
...output omitted..

```
