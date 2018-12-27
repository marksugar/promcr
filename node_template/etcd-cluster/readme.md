

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
      - '172.25.50.16:2379'
      - '172.25.50.17:2379'
      - '172.25.50.18:2379'
      labels:
        group: 'etcd' 
```        
