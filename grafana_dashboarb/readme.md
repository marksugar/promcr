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
