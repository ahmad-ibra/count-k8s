# count-k8s
- [Server](https://github.com/ahmad-ibra/count-server)
- [Client](https://github.com/ahmad-ibra/count-client)

## Usage
Create a local cluster
```
❯ kind create cluster
```

Run the helm chart
```
❯ helm install count-k8s ./count-k8s -n count --create-namespace
```
