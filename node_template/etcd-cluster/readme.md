
```
 iptables -I INPUT 5 -p tcp -m tcp -m state --state NEW -m multiport --dports 18880,9100,2379 -m comment --comment "etcd" -j ACCEPT
 ```
配置文件定义：

```
  - job_name: 'etcd'
    metrics_path: /metrics
    scheme: https
    tls_config:
      cert_file: 'ssl/server.pem'
      key_file: 'ssl/server-key.pem'
      insecure_skip_verify: true
    static_configs:
    - targets: 
      - '172.25.90.116:2379'
      - '172.25.90.117:2379'
      - '172.25.90.118:2379'
      labels:
        group: 'etcd' 
```        
延伸阅读:

- https://coreos.com/etcd/docs/latest/metrics.html
- https://etcd.readthedocs.io/en/latest/operate.html#v3-3
- https://github.com/grpc-ecosystem/go-grpc-prometheus#useful-query-examples
