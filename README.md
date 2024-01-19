# Kubernetes User Creation

In this guide, we will find out how to create a new user in Kubernetes. Will be using On-Premise Kubernetes for the demonstration.


1: Creating User in Linux

Creation of the user in linux will help the actal user to enter on the server's Master node  then that will be mapped with the user in Kubernetes.

```
useradd adarsh
```

1: Generate certificates for the user.

Following command will generates an Ed25519 private key using OpenSSL.

```
openssl genpkey -out adarsh.key -algorithm Ed25519
```
After executing this command, OpenSSL will generate an Ed25519(which is a modern elliptic curve cryptography algorithm) private key and save it in the “john.key” file. 

```
openssl req -new -key adarsh.key -out adarsh.csr -subj "/CN=adarsh,/O=edit"
```


2: Create a certificate signing request (CSR).

For request we have to obtain the base64-encoded representation of the contents of the “john.csr” file without newline characters. This encoding is often useful for transmitting or storing binary data in a text-based format.

```
cat john.csr | base64 | tr -d "\n"
```
This will produce output like  -  LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlHZU1GSUNBUUF3SHpFT01Bd0dBMVVFQXd3RmFtOW9iaXd4RFRBTEJnTlZCQW9NQkdWa2FYUXdLakFGQmdNcgpaWEFESVFEbk9nYXEzSEcyMkw1dFROZ0JaQ1lHYkJiSjdXQ05BTmE2aWhsNll0NWFkNkFBTUFVR0F5dGxjQU5CCkFPeVJaZlpKY2c3eE90eHBjUmFqZmlySk9WcVkvaE9CQldneEJubERFek4rbXpJRm12MkU5czNoaXBQZjBOYk8KMkNrR0pNR0NhOXJabStVRHowelhqZ009Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=


Then we will copy a CertificateSigningReques template from k8 docs. Simply copy the output from previous command & replace in .spec.request.

```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: adarsh
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlHZU1GSUNBUUF3SHpFT01Bd0dBMVVFQXd3RmFtOW9iaXd4RFRBTEJnTlZCQW9NQkdWa2FYUXdLakFGQmdNcgpaWEFESVFEbk9nYXEzSEcyMkw1dFROZ0JaQ1lHYkJiSjdXQ05BTmE2aWhsNll0NWFkNkFBTUFVR0F5dGxjQU5CCkFPeVJaZlpKY2c3eE90eHBjUmFqZmlySk9WcVkvaE9CQldneEJubERFek4rbXpJRm12MkU5czNoaXBQZjBOYk8KMkNrR0pNR0NhOXJabStVRHowelhqZ009Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF

```


3: Sign the certificate using the cluster certificate authority.

```
kubectl get csr
```

```
kubectl certificate approve adarsh
```

4: Create a configuration specific to the user.

5: Add RBAC rules for the user or their group.





