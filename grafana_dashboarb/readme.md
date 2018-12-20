## add plugins
```
[root@Node /data/prometheus]# docker exec -it grafana sh
$ grafana-cli plugins install grafana-piechart-panel
installing grafana-piechart-panel @ 1.3.3
from url: https://grafana.com/api/plugins/grafana-piechart-panel/versions/1.3.3/download
into: /var/lib/grafana/plugins

✔ Installed grafana-piechart-panel successfully 

Restart grafana after installing plugins . <service grafana-server restart>
```
```
docker-compose -f ./docker-compose.yaml up -d
```

## 注意
此处的模板都是基础模板，包含cadvisor，node_exporter，以group区分

模板中的group对应配置文件中的__meta_consul_tags字段，如下：

```
        - source_labels: ['__meta_consul_tags']
          regex:         ',(prometheus|app),'
          target_label:  'group'
          replacement:   '$1'
```		  
compose的标签在这里转成group