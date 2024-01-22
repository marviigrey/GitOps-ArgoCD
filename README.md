This Repository describes my learning process on the CICD tool called ArgoCD. I created this repo to share all that i have learnt about the CICD tool called argoCD and how it can be integrated with other tools.

Pre-requisite:
 1. Basic understanding of Continuous integration and the concept of continuous deployment.
 
 2. We will be working round the kubernetes eco-system,ensure you have kubernetes installed on your machine.

 3. ArgoCD also installed.

Steps: 
    - Install argo using the helm chart in the github-repo:
            kubectl create namespace argocd
            kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.9.3/manifests/install.yaml
    
    - change argocd-server service to type NodePort:
            kubectl edit svc -n argocd argo-server
            #replace ClusterIP to NodePort or Loadbalancer

    - After changing the service type, we now have our argocd UI accessible  through the internet.
    - Next is to get the custom password and replace it with any password of our choice, run:

               kubectl -n argocd get secrets argocd-initial-admin-secret -o json | jq .data.password -r | base64 -d
               
    - This command will help to decode the encoded password stored in a kubernetes resources called secrets. 

    - Access the UI through the argoCD service Endpoint,by running:
    
                kubectl get svc -n argocd

    - Install the argoCD CLI:
                #Download the package from the official github project repo:
                wget https://github.com/argoproj/argo-cd/releases/download/v2.9.3/argocd-linux-amd64

                #rename the package:

                mv argocd-linux-amd64 argocd

                chmod +x argocd

                sudo mv argocd /usr/local/bin/

    - Test argocd cLI to login:

                argocd login <service-endpoint-of-argocd-server.> -y

                argocd app list

                argocd cluster list

ArgoCD Application Project:

An ArgoCD Application can be created using a YAML definition file. This file contains the source of the  application  which can be the github repo where the codes are stored in, destination which is the kubernetes namespace, and the sync Details which refers to the synchronization process that ArgoCD manages between a desired state (defined in Git) and the actual state of applications running in your Kubernetes cluster.

For this project we will be creaating a simple test project that contains kubernetes resource manifest.  The resources can be found [here](https://github.com/marviigrey/GitOps-ArgoCD/tree/main/k8s-resources).

We can create the app using the cli or the UI. For this project, i will be using the cli to create the app.

To create an argoCD app run:
        argocd app create <app name> --repo <github-repo> --path <path to manifest-file> -- dest-namespace <namespace> --dest-server <cluster address>.

Argocd Project: 
 An ArgoCD project is a logical grouping or namespace within ArgoCD that helps in organizing and managing a set of related applications, repositories, and resources.

 We can easily create an argocd project on our argocd UI. 

 Reconcilation Loop: this is a state of synchronization of your git repository to your kubernetes cluster. This loop ensures both the git repository containing manifests that were used to create resources in our kubernetes cluster remains the same with that of the cluster. Automatically, Argocd makes a pull request with or without a timeout to ensure both states are synchronized. We can remove a polling delay by setting up a webhook on github. This can be done by adding the server address of our argocd. 

 Webhook: argoCD also support github webhook for automatic pulling of changes in our code repository.

 Application health checks status and their meaning: 
 All resources are 100% healthy - Healthy
 Resources are unhealthy, but could still be healthy given time. - Progressing.
 Resource status indicates a failure or inablity to reach a healthy state - Degraded.
 Resource is not present in the cluster - Missing.
 Resource is suspended or paused, typical example is a paused deployment - Suspended.
 Health assessment failed and actual health status is unknown - Unknown.
 
Trying to configure custom health checks using configMaps in argoCD, by adding data into 
the configmap deployed during argoCD configuration. This custom health checks are used in 
a situation where we want to check our application health for specific resources.

Synchronisation strategies: ArgoCD allows customization on application synchronisation between the state of git and that of the kuberentes cluster.
Automatic Sync: ArgoCD will apply the changes automatically by updatig the resources in the target clusters.
Manual Sync: changes are detected by argoCD but resources are not updated automatically.
Auto-pruning: Thiis feature describes what happens when files are deleted or removed from git.
Self Heal: Self heal defines what argoCD does when you make "kubectl edit" changes directly in the cluster.

You can set any of the above mentioned strategy to synchronize the state of your git repository with the state of your kubernetes cluster.

ArgoCD resources can also be created using manifest files, meaning that the resources we create using the argoCD UI or CLI can also be created using manifest files. This aproach is known as the declarative approach. The manifest file contains the namespace,argoCD project, RepoURL, path to the resources in the repo, Synchronisation policies and many more.
We can simply run:
        
        kubectl apply -f <path to manifest>

This automatically create all resources listed in the declarative script.

The App of Apps pattern enables us to progamatically and automatically generate ArgoCD applications. 

Deploying aplications using Helm Charts: We can deploy applications to argocd using custom helm charts stored in git repositories, bitnami, or any package manager. After deploying applications using argocd from a helm chart, you wont see the applications when you run "helm app list" but rather view them on argocd ui or "argocd app list" in the cli. 
Aside the helm chart being a popular package manager, we also make use of bitnami a package manager used for deploying resources both on VMs and a k8s cluster. We can use this package manager to install apps running in our cluster by connecting the bitnami helm chart repo to our argoCD repo lists.

Multi-cluster Deployments: ArgoCD also supports multi-cluster deployment.
To do this: we will need to set config and add the cluster:

        kubectl config set-clsuter prod --server=https://1.2.3.4 --certificate-authority=prod.crt 
        kubectl config set-credentials admin --client-certificate=admin.crt --client-key=admin.key
        kubectl config set-context admin-prod

Once you have multiple cluster configured, you can register your external cluster using this command:

        argocd cluster add <context-name>

Doing this, resources such as secrets will be created and registered under argoCD. When you open your application you will be able to see the cluster registered as a destination.

Role based access control policies in argoCD: 
In argoCD we can control users and groups access to our clusters using SSO products such as dex octa or simply using local user management system provided by argocd:

        kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"accounts.ali": "apiKey,login"}}'

This command creates a new user and sets an api token, which allows generating of JWT JSON web token authentification for API access whereas the login capability allows the user to login using the User-Interface.

You can also set up users through argoCD CLI:
        argocd account update-password --account <account-name>

We can also modi assigning custom roles to users by editing the argocd config map using patch imperative command.

Dex Okta Connector: Dex is an identity manager that powers authentication for other applications by using OpenID connect. Note: i made use of Auth0 because it's free and it could do the demo that i wanted.
#client-Id RBPzLnjBltOvlXDVrMN9Kzz0mO3gudOw
#client-secret 3c-nxh3eEKqUe8e8TJf1tBvNdAEuTKwl98l1CyI7TzkaxD9Kp-B7o82aRpbEuXNY
In the auth, i created an organisation and two users, i added them into the organisation.
To register your argocd application on argoCD:

    Take note of the clientId and clientSecret values.
    Register login url as https://your.argoingress.address/login
    Set allowed callback url to https://your.argoingress.address/auth/callback
    Under connections, select the user-registries you want to use with argo.

To configure oidc for argoCD run:
        
        kubectl edit configmaps -n argocd argocd-cm
 #add this to the configmap and edit the values       
data:
  application.instanceLabelKey: argocd.argoproj.io/instance
  url: https://your.argoingress.address
  oidc.config: |
    name: Auth0
    issuer: https://<yourtenant>.<eu|us>.auth0.com/
    clientID: <theClientId>
    clientSecret: <theClientSecret>
    requestedScopes:
    - openid
    - profile
    - email
    # not strictly necessary - but good practice:
    - 'http://your.domain/groups'
...

To set RBAC for users,groups, run:

          kubectl edit configmap argocd-rbac-cm
        #paste this on the cm and edit the values:
        ...
data:
  policy.csv: |
    # let members with group someProjectGroup handle apps in someProject
    # this can also be defined in the UI in the group-definition to avoid doing it there in the configmap
    p, someProjectGroup, applications, *, someProject/*, allow
    # let the group membership argocd-admins from OIDC become role:admin - needs to go into the configmap
    g, argocd-global-admins, role:admin
  policy.default: role:readonly
  # essential to get argo to use groups for RBAC:
  scopes: '[http://your.domain/groups, email]' 
...

Sealed-Secrets: When running applications on our cluster, sometimes we have sensitive details we want to keep as secrets. We make use of bitnamy sealed secret helm chart.  When we install the binami sealed secret chart using helm, we also need to software for encrypting kubernetes secrets such as kube-seal. Kube-seal CLI is used for encrypting secrets in kubernetes.
 
To install bitnami sealed secrets:
        1. go to the argocd ui and add the bitnami helm chart repo for installing apps.
        2. on the type of repo,use the helm type.
        3. for the chart, select or search the "sealed-secrets" chart.
        4. install it on the kube-system namespace on your cluster.
        5. Deployments will be created and pods will be initialized.

After that we install the kubeseal CLI:
       - wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.25.0/kubeseal-0.25.0-linux-amd64.tar.gz
       - tar -xvf kubeseal-0.25.0-linux-amd64.tar.gz
       - chmod +x kubeseal
       - mv kubeseal /usr/local/bin
       - kubeseal --version

Once we have the kubeseal installed, we move further to test the kubeseal with a dummy secret. what the kubeseal will do when it discovered the secret is that it will encrypt it then store it.
