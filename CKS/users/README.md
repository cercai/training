# Users, ServiceAccounts and Groups

Even though a normal user cannot be added via an API call, any user that presents a valid certificate signed by the cluster's certificate authority (CA) is considered authenticated. In this configuration, Kubernetes determines the username from the common name field in the 'subject' of the cert (e.g., "/CN=bob"). From there, the role based access control (RBAC) sub-system would determine whether the user is authorized to perform a specific operation on a resource.

The purpose of this tutorial is to create the user Tom.

## Create the private key

Let's start by creating a private key for Tom in  Privacy-Enhanced Mail (PEM) format.
```console
$ openssl genrsa -out tom-key.pem 2048
```

## Create a Certificate Signing Request(CSR)

Then create the csr for user Tom:
```console
$ openssl req -new -subj "/CN=tom" -key tom-key.pem  -out tom-csr.pem
```
In the subject, we see that in the *CN* field there is the username.
This will output the file *tom-csr.pem* that will be used in the next step.

## Create a user

Create the user CertificateSigningRequest manifest
```
cat > tom-csr.yaml <<EOF
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: tom
spec:
  request: $(cat tom-csr.pem | base64 | tr -d '\n')
  signerName: example.com/serving
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```

Ten send it to the cluster api server with the command `kubectl apply -f tom-csr.yaml`.
Verify the cluster has received the csr with `kubectl get csr`
```cmd
NAME   AGE   SIGNERNAME            REQUESTOR          REQUESTEDDURATION   CONDITION
tom    2s    example.com/serving   kubernetes-admin   <none>              Pending
```

Before approving or declining the certificate, check it with `kubectl describe csr tom`.
Then approve/deny it
```
$ kubectl certificate approve tom
```

If you deny it by mistake, you must delete the csr before sending the csr again to the api-server.
```
# If you have chosen to deny it
$ kubectl certificate deny tom

# You must delete the csr
$ kubectl delete csr tom

# before sending the csr again to the api-server
$ kubectl apply -f tom-csr.yaml

# then approve it
$ kubectl certificate approve tom
```

Verify the cluster has received the csr with `kubectl get csr`
```cmd
NAME   AGE   SIGNERNAME            REQUESTOR          REQUESTEDDURATION   CONDITION
tom    11s   example.com/serving   kubernetes-admin   <none>              Approved
```
