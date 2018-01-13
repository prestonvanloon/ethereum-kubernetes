# Ethereum on Kubernetes

A simple test network running on kubernetes.

## Requirements

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- A kubernetes cluster via [minikube](https://github.com/kubernetes/minikube) or your favorite cloud provider.
- A basic understanding of Ethereum networks.

## Deploying

First, we'll create the secrets used by our containers. 

```bash
kubectl create secret generic bootkey1 --from-file=boot.key
kubectl create secret generic genesis --from-file=genesis.json
```

Second, start up a bootnode.

```bash
kubectl apply -f bootnode.yaml
```

You can verify the bootnode is online with `kubectl get pods`.

```
NAME                         READY     STATUS    RESTARTS   AGE
bootnode1-3661432000-2djzn   1/1       Running   0          1m
```

Now we'll launch the eth stats page, node, and miner

```bash
kubectl apply -f ethstats.yaml
kubectl apply -f node.yaml
kubectl apply -f miner.yaml
```
Verify those pods are ready with `kubectl get pods`

```
NAME                         READY     STATUS    RESTARTS   AGE                         
bootnode1-3661432000-2djzn   1/1       Running   0          4m                         
ethstats-3596783629-7412f    1/1       Running   0          1m                         
miner-4221110532-2qb5m       1/1       Running   0          1m                          
node-2707557160-4866r        1/1       Running   0          1m    
```

Your local network is running!

## Interacting with the network

### Viewing eth stats 

To view eth status, set up a port forward from the pod to your machine.

```bash
kubectl port-forward ethstats-3596783629-7412f 3000:3000
```

Then you should be able to access it at localhost:3000 in your browser.

### JSON RPC 

To interact with a node via RPC, set up a port forward from the pod to your machine.

```bash
kubectl port-forward node-2707557160-4866r 8545:8545
```

Then you should be able to access it on localhost:8545.

### Geth client

To interact with a node or miner via geth client, run the following

```bash
kubectl exec -it miner-4221110532-2qb5m -- geth attach /ethereum/geth.ipc
```

## Scaling the deployment

Running a single node and a single miner is great, but if you'd like to test multiple and you have
capacity in your cluster then run the following.

```bash
kubectl scale deployment miner --replicas=3
kubectl scale deployment node --replicas=8
```

## TODO

- Add a job to simulate transactions
- Add an all-in-one script

## Notes

The resource limit for the miner is relatively low, this means it will take ~20 minutes
to generate the DAG. Change those values in the yaml file if you'd like to spend more
resources mining. 
