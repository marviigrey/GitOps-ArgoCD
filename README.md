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

