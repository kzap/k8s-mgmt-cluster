# Local KinD Kubernetes Setup w/ ArgoCD

This sets up a local Kubernetes cluster using KinD (Kubernetes in Docker) and installs ArgoCD and uses it to sync the following applications onto the cluster:

- argocd (argocd syncs itself!)
- nginx-ingress
- cert-manager
- linkerd

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