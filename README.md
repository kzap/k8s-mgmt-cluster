# Local KinD Management Cluster w/ ArgoCD

This sets up a local Kubernetes cluster using KinD (Kubernetes in Docker) and installs ArgoCD and uses it to sync the following applications onto the cluster:

- argocd (argocd syncs itself!)
- nginx-ingress
- cert-manager
- linkerd
- crossplane

It then allows you to create AWS resources using Crossplane such as an entire EKS cluster as shown below.

## Getting Started

- Make sure you have KinD installed:

    ```sh
    brew install kind
    ```

- Create the KinD cluster from the config:

    ```sh
    kind create cluster --config ./kind/v1.20-config.yaml
    ```

- Install ArgoCD using `kustomize` so it can fetch all the other applications

    ```sh
    kubectl apply -k ./argocd
    ```

- Create `secret` to contain AWS Credentias (these credentials will have to be close to Admin as you need to create many AWS resources)

    ```sh
    # Replace `[...]` with your access key ID`
    export AWS_ACCESS_KEY_ID=[...]

    # Replace `[...]` with your secret access key
    export AWS_SECRET_ACCESS_KEY=[...]

    # Create a temporary file to hold the credentials
    echo "[default]
    aws_access_key_id = $AWS_ACCESS_KEY_ID
    aws_secret_access_key = $AWS_SECRET_ACCESS_KEY
    " > ./aws-creds.conf

    # Create a Kubernetes secret using this file
    kubectl --namespace crossplane-system \
        create secret generic aws-creds \
        --from-file creds=./aws-creds.conf

    # Delete this temp file
    rm ./aws-creds.conf
    ```

- Create a AWS EKS Cluster using Crossplane by applying the `eks-cluster.yaml` or `eks-cluster-medium.yaml`. This is a slightly modified version of @vfarcic's code found [here](https://github.com/vfarcic/devops-toolkit-crossplane/tree/master/crossplane-config).

    ```sh
    kubectl apply -f ./crossplane-eks-cluster/eks-cluster.yaml
    kubectl apply -f ./crossplane-eks-cluster/eks-cluster-medium.yaml
    ```

- Observe the creation of the resources:

    ```sh
    kubectl get managed,compositecluster,clusterclaim
    ```