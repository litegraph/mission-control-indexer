# FAQ

## Resources

### What system resources are required to participate in the testnet?

![Resource Guidance](files/infrastructure-resources.png)

### What are the system requirements for Prometheus and Grafana?

- Prometheus doesn't need a lot of CPU. It may require up to ~1.5GB of memory
  in later phases. 200GB should be more than enough disk space for it.
- Grafana needs even less CPU, <256MB of memory and no more than 1GB of disk
  space.

### What kind of Ethereum node is required for indexing?

- A mainnet archive node will be enough to participate in the testnet.
- Ideal would be a mainnet archive node with traces. We will include
  subgraphs that require that in all phases, only those with access to the
  OpenEthereum `trace` API will be able to index these.
- Also keep in mind that for phase 0, syncing an Ethereum node may take too
  long, so plan accordingly.
- Note: Using just a single Ethereum node may in later phases limit the
  number of subgraphs you can index (due to the required I/O throughput). We
  recommend load balancing across a few nodes.

### How to configure Prometheus to scrape Graph Node metrics?

https://prometheus.io/docs/prometheus/latest/installation/ has some
instructions on installing Prometheus, as well as links to binaries. Once
Prometheus is up and running, the main difference compared to k8s is how you
configure it so that it scrapes the Graph node metrics.

Prometheus gets its metrics using so-called "scrape configs".
https://github.com/graphprotocol/mission-control-indexer/blob/master/k8s/base/prometheus.yaml#L103-L128
shows the config file using a scrape job with kubernetes_sd_configs. If you
run everything bare metal, you'll probably need to use a job with
static_configs (pointing to e.g. http://localhost:8040/ if that's the port
that your graph-node serves the metrics from). Check out
https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config
and
https://prometheus.io/docs/prometheus/latest/configuration/configuration/#static_config
in particular.

## Phase 0

### What does the `ingress.kubernetes.io/target-proxy` in the k8s setup mean?

That is only useful if you want to set up your ingress so it's accessible
through Google's global HTTPS load balancer. We found that to be the easiest
way to expose an ingress via an auto-updated SSL certificate. Without that,
all you get is a static IP address (which you can then still add a DNS entry
for, but it'll be HTTP). Which may be fine, depends on how far you want to
take it.

If you are interested in the HTTPS setup, here's a quick writeup:
https://jannis.github.io/notes/google-cloud-https.

### How do I deploy phase 0 subgraphs to the Graph nodes?

**Hint:** Don't build them from the subgraph repos. That will lead to
different subgraph deployment IDs than the ones required for phase 0.

If you are using the k8s setup, all you have to do is e.g.:

```bash
# 1. Create the subgraph name
kubectl exec shell -- create "molochventures/moloch"

# 2. Deploy the expected version
kubectl exec shell -- deploy "molochventures/moloch" "QmTXzATwNfgGVukV1fX2T6xw9f6LAYRVWpsdXyRWzUR2H9" "index_node_0"
```

Otherwise it would be something like (using [HTTPie](https://httpie.org/)):

```bash
# 1. Create the subgraph name
http post <GRAPH_NODE>:8020 \
  jsonrpc="2.0" \
  id="1" \
  method="subgraph_create" \
  params:='{"name": "molochventures/moloch"}'

# 2. Deploy the expected version
http post <GRAPH_NODE>:8020 \
  jsonrpc="2.0" \
  id="1" \
  method="subgraph_deploy" \
  params:='{"name": "molochventures/moloch", "ipfs_hash": "QmTXzATwNfgGVukV1fX2T6xw9f6LAYRVWpsdXyRWzUR2H9", "node_id": "<target-node-id>"}'
```

## Phase 1

### Can I reregister my indexer on chain?

Yes, of course. All you have to do is:

1. Change the registration parameters, e.g. the `--public-indexer-url` or
   `INDEXER_AGENT_PUBLIC_INDEXER_URL` that are passed to the indexer agent.
2. Restart the indexer agent. It should automatically re-register.

### What happened to my indexing metrics?

This [graph-node pull
request](https://github.com/graphprotocol/graph-node/pull/1826) changes the
names of a lot of Prometheus metrics. We used to use the deployment
identifier `Qm..` in the name of Prometheus metrics, but that was
undesirable for two reasons:

- for indexers that have a large number of deployments, UI tools become
  unresponsive because of the resulting large number of metrics
- it is next to impossible to aggregate such metrics across all deployments

The change in the pull request makes the deployment identifier a label
`deployment` on all such metrics, and also replaces any mention of the word
`subgraph` in the metric name with `deployment` to be consistent with the
general nomenclature in the Graph Protocol ecosystem. Once you deploy a
`graph-node` version that includes that pull request, the system will no
longer update metrics with the old names, even for already existing
subgraph deployments.

The [`grafana`](./grafana) directory contains updated dashboards. Before
importing them into your existing Grafana installation, it is highly
recommended that you make a copy of the old dashboard withing Grafana if
you want to view information about already indexed deployments which use
the old metric names. The Grafana [kubernetes](./k8s/base/grafana.yaml)
container has also been updated with dashboards using the new naming
convention.

## Phase 2

### What IP addresses is the query traffic coming from?

This is the list. If you have a firewall set up, make sure to allow these IP
addresses to send requests to your indexer service.

```
35.247.236.236
34.95.191.237
35.198.62.223
35.198.38.97
35.199.76.174
34.95.189.37
35.199.106.49
34.87.223.32
35.201.30.225
35.197.165.223
35.244.123.62
35.201.27.80
35.201.23.133
35.189.13.40
35.246.241.146
35.198.180.221
35.246.140.28
34.89.192.36
34.89.135.245
35.246.171.70
34.107.25.111
35.228.53.98
35.228.254.44
35.228.8.135
35.228.174.24
35.228.111.115
35.228.148.146
35.228.235.187
35.221.25.192
35.194.79.231
35.245.100.101
35.188.231.6
35.245.185.85
35.221.32.16
35.236.218.116
34.121.194.234
35.223.66.230
35.184.47.137
35.239.16.123
34.70.195.215
35.232.100.126
35.222.176.113
34.105.13.193
35.233.207.100
35.199.160.253
35.247.75.108
35.247.24.198
35.230.90.221
35.230.69.145
34.85.90.236
34.84.113.138
34.84.165.63
34.84.154.117
35.243.118.229
34.84.103.92
34.84.248.234
34.93.235.38
34.93.217.11
34.93.247.76
34.93.3.171
35.244.26.219
34.93.193.114
34.93.238.6
35.220.131.196
35.220.139.150
34.92.75.78
34.96.232.37
34.96.224.43
35.220.173.188
34.96.169.212
```
