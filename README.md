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

