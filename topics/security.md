# Securing access to cluster

The Kubernetes cluster has a built-in CSR signing capability.  Signed certificates by the
Kubernetes control plane are the basis for authentication and authorization.

User provides a CSR with username and group membership information in the certificate signing
request subject.  Kubernetes administrator approves the CSR signing request to grant the access
to the specific Kubernetes cluster.

General steps:
- User creates SSL private key
- User create CSR with desired subject (starts with /, field separated by /)
    - CN := username
    - O := groups
- Post CSR resource to cluster
- Approve CSR in cluster
- Extract signed certificate from cluster
- Create contexts (stored in local .kube/config)
    - user (embed-certs)
    - context (bound to cluster and user)
- Create cluster role with rules
    - use existing one to get template
    - get list of API groups via curl to API server (/etc/kubernetes/pki files needed)
    - sudo curl --cacert ca.crt --cert apiserver-kubelet-client.crt --key apiserver-kubelet-client.key IP:6443
    - apiGroups, resources, verbs, nonResourceURLs, verbs (all can be discovered by curl to apiGroup endpoint)
- Create cluster role binding (to role and group)
    - "network-admin" to match group ("network-admin" stored in authn cert passed in request)

Specifics
- OpenSSL
    - Create key via genrsa (2048 bits)
    - Create request via req
- Kubernetes
    - Create resource for CertificateSigningRequest
        - No 'kubectl create csr' template assistance
        - kubectl get csr will list existing CSRs for the cluster and can use as template
        - Better to memorize the usages
            - digital signature
            - key encipherment
            - client auth
        - The base64 inline encoding has challenges on Ubuntu and MacOS
            - https://stackoverflow.com/questions/65846911/kubectl-trying-to-create-certificatesigningrequest-error-error-from-server-bad
            - So base encode, truncate the new lines and paste the string manually into the csr YAML
    - Approve the CSR (k cert approve NAME)
    - Extract the certificate (k get csr NAME -o jsonpath='{.status.certificate}') and base64 decode it
    - Create user in kubernetes with the certificate
        - kubectl config set-credentials tim --client-certificate=x.crt --client-key=x.key --embed-certs
    - Create network-admin context
        - kubectl config set-context network-admin --cluster=X --user=Y
    - Change context to network-admin
        - kubectl config use-context network-admin
    - Change context back to cluster admin
        - kubectl config use-context kubernetes-admin@kubernetes