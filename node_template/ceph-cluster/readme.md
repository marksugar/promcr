在此处的compose文件中有添加了ceph，如下：
```
  ceph_exporter:
    container_name: ceph_exporter
    image: digitalocean/ceph_exporter:2.0.1-luminous
    network_mode: "host"
    volumes:
      - /etc/ceph:/etc/ceph
    ports:
      - 9128:9128
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    cpus: '0.15'
    mem_limit: 50M    
    labels:
      SERVICE_TAGS: ceph-cluster
```

你可能也需要将配置文件修改
```
  - job_name: 'ceph_exporter'
    metrics_path: /metrics
    scheme: http
    consul_sd_configs:
      - server: 127.0.0.1:8500
        services: ['ceph_exporter']
    relabel_configs:
        - source_labels: ['__meta_consul_service']
          regex:         '(.*)'
          target_label:  'job'
          replacement:   '$1'
        - source_labels: ['__meta_consul_service_address']
          regex:         '(.*)'
          target_label:  'instance'
          replacement:   '$1'
        - source_labels: ['__meta_consul_service_address', '__meta_consul_service_port']
          regex:         '(.*);(.*)'
          target_label:  '__address__'
          replacement:   '$1:$2'

        - source_labels: ['__meta_consul_tags']
          regex:         ',(ceph-cluster|cephfs),'
          target_label:  'group'
          replacement:   '$1'
```
其中ceph-cluster|cephfs是容器的标签，也对应此处的模板

## deploy
```
curl -Lk https://raw.githubusercontent.com/marksugar/promcr/master/node_template/ceph-cluster/docker-compose-ceph.yaml -o /opt/docker-compose.yaml
docker-compose -f /opt/docker-compose.yaml up -d
```

