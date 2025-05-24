
# Create the namespaces

Create the manifest for a namespace
```console
$ kubectl create namespace my-namespace --dry-run=client -o yaml > namespaces.yaml
```

The file `namespaces.yaml` must now look like this:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: my-namespace
spec: {}
status: {}
```

Remove the useless parts: *creationTimestamp: null*, *spec: {}* and *status: {}*.

We will now duplicate the bloc twice to create three namespaces named *dev*, *qual* and *prod*.
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: qual
---
apiVersion: v1
kind: Namespace
metadata:
  name: prod
```

Then apply the file to create the 3 namespaces with the command `kubectl apply -f namespaces.yaml`
```
namespace/dev created
namespace/qual created
namespace/prod create
```
