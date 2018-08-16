# How to use helm to install elasticsearch-curator

### 工具介绍 

```
Elasticsearch Curator helps you curate, or manage, your Elasticsearch indices and snapshots by:
1.Obtaining the full list of indices (or snapshots) from the cluster, as the actionable list
2.Iterate through a list of user-defined filters to progressively remove indices (or snapshots) from this actionable list as needed.
3.Perform various actions on the items which remain in the actionable list. 
```
###  参考资料

```	
https://github.com/helm/charts/tree/master/incubator/elasticsearch-curator
https://github.com/elastic/curator
```

###  设置私有helm repo

```
#helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
#helm fetch incubator/elasticsearch-curator --version 0.4.0
#helm push elasticsearch-curator-0.4.0.tgz apphub

```

### 自定义values文件

```
#vi logcenter-dev.yaml
#helm install --dry-run --debug --name logcenter-dev -f logcenter-dev.yaml  apphub/elasticsearch-curator --version 0.4.0
```

### 安装elasticsearch-curator服务

```
#helm install --name logcenter-dev -f logcenter-dev.yaml  apphub/elasticsearch-curator --version 0.4.0

NAME:   logcenter-dev
LAST DEPLOYED: Thu Aug  9 20:25:08 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                                        DATA  AGE
logcenter-dev-elasticsearch-curator-config  2     0s

==> v1beta1/CronJob
NAME                                 SCHEDULE   SUSPEND  ACTIVE  LAST SCHEDULE  AGE
logcenter-dev-elasticsearch-curator  0 1 * * *  False    0       <none>         0s


NOTES:
A CronJob will run with schedule 0 1 * * *.

The Jobs will not be removed automagically when deleting this Helm chart.
To remove these jobs, run the following :

    kubectl -n logging delete job -l app=elasticsearch-curator
```

### 查看elasticsearch-curator服务

```
# kubectl get cronjob 
NAME                                      SCHEDULE    SUSPEND   ACTIVE    LAST SCHEDULE   AGE
logcenter-dev-elasticsearch-curator       0 1 * * *   False     0         <none>
```

### curator_cli查看服务

```
#pip install elasticsearch-curator
#pip install --upgrade pip
# curator_cli --host 10.72.244.73 --port 9200 show_indices --verbose
.kibana                            open   48.8KB        6   1   1 2018-07-19T02:44:42Z
.monitoring-es-6-2018.08.02        open  336.4MB   590670   1   1 2018-08-02T00:00:06Z
.monitoring-es-6-2018.08.03        open  338.6MB   591630   1   1 2018-08-03T00:00:00Z
.monitoring-es-6-2018.08.04        open  333.9MB   590196   1   1 2018-08-04T00:00:01Z
.monitoring-es-6-2018.08.05        open  340.8MB   590366   1   1 2018-08-05T00:00:02Z
.monitoring-es-6-2018.08.06        open  332.7MB   592188   1   1 2018-08-06T00:00:00Z
.monitoring-es-6-2018.08.07        open  333.0MB   590208   1   1 2018-08-07T00:00:01Z
.monitoring-es-6-2018.08.08        open  329.8MB   590028   1   1 2018-08-08T00:00:02Z
.monitoring-es-6-2018.08.09        open  229.8MB   339400   1   1 2018-08-09T00:00:03Z
.monitoring-kibana-6-2018.08.02    open   12.3MB    51830   1   1 2018-08-02T00:00:02Z
.monitoring-kibana-6-2018.08.03    open   12.4MB    51828   1   1 2018-08-03T00:00:04Z
.monitoring-kibana-6-2018.08.04    open   12.2MB    51830   1   1 2018-08-04T00:00:00Z
.monitoring-kibana-6-2018.08.05    open   12.2MB    51830   1   1 2018-08-05T00:00:00Z
.monitoring-kibana-6-2018.08.06    open   12.4MB    51832   1   1 2018-08-06T00:00:00Z
.monitoring-kibana-6-2018.08.07    open   12.4MB    51830   1   1 2018-08-07T00:00:02Z
.monitoring-kibana-6-2018.08.08    open   12.3MB    51830   1   1 2018-08-08T00:00:00Z
.monitoring-kibana-6-2018.08.09    open    7.1MB    29070   1   1 2018-08-09T00:00:01Z
.monitoring-logstash-6-2018.08.02  open  311.8MB  2228260   1   1 2018-08-02T00:00:03Z
.monitoring-logstash-6-2018.08.03  open  306.9MB  2228088   1   1 2018-08-03T00:00:02Z
.monitoring-logstash-6-2018.08.04  open  298.0MB  2228088   1   1 2018-08-04T00:00:01Z
.monitoring-logstash-6-2018.08.05  open  297.5MB  2228260   1   1 2018-08-05T00:00:00Z
.monitoring-logstash-6-2018.08.06  open  293.8MB  2228174   1   1 2018-08-06T00:00:00Z
.monitoring-logstash-6-2018.08.07  open  289.3MB  2228174   1   1 2018-08-07T00:00:03Z
.monitoring-logstash-6-2018.08.08  open  279.4MB  2228174   1   1 2018-08-08T00:00:00Z
.monitoring-logstash-6-2018.08.09  open  160.1MB  1249838   1   1 2018-08-09T00:00:00Z
logstash-2018.07                   open    1.2MB      454   5   1 2018-07-25T03:35:39Z
logstash-2018.08                   open  453.6MB   261194   5   1 2018-08-01T06:45:47Z
tomcat-log-2018.07                 open   43.4MB   203850   5   1 2018-07-25T03:35:39Z
tomcat-log-2018.08                 open   19.3MB    89372   5   1 2018-08-01T12:34:35Z

#curator_cli --host 10.72.244.73 --port 9200 delete_indices  --filter_list '[{"filtertype":"age","source":"creation_date","direction":"older","unit":"days","unit_count":3},{"filtertype":"pattern","kind":"prefix","value":".monitoring-es-6-|.monitoring-kibana-6-|.monitoring-logstash-6-"}]'
```

