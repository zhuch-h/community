# KubeSphere Monitoring

This documentation contains backend development guide for interaction with Prometheus. The monitoring backend provides the capabilities of:

- Metrics query
- Metrics sorting
- Multi-tenant isolation

## File Tree

The listing below covers all folders related to the monitoring backend.

```yaml
/pkg
  ├─api
  │  └─monitoring         # declare structs for api responses
  │     └─v1alpha2
  ├─apiserver             # implement handler for http requests
  │  └─monitoring
  ├─kapis                 # register APIs and routing
  │  └─monitoring
  │     ├─install
  │     └─v1alpha2
  ├─models
  │  └─metrics
  │     ├─constants.go
  │     ├─metrics.go       # proxy prometheus metrics query
  │     ├─metrics_rules.go # promql expressions for builtin metrics
  │     ├─namespaces.go    # append metric info to namespace resource request
  │     ├─types.go
  │     └─util.go          # metrics sorting
  └─simple
     ├─factory.go          # factory functions for prometheus client options
     └─client
        └─prometheus
           ├─options.go    # prometheus client options
           └─prometheus.go # prometheus client code
```

## API Design

To support multi-tenant isolation, the monitoring backend proxies Prometheus query requests. KubeSphere's APIs have the format like below:

```bash
GET /namespaces/{namespace}/pods
GET /namespaces/{namespace}/pods/{pod}
```

KubeSphere API gateway will decode the URL and conduct authorization. A person who doesn't belong to a namespace will be refused to make a request. Besides, note that the two examples above have slightly different meanings. The first is to retrieve all pod-level metrics in the namespace, while the latter is for a specific pod.
