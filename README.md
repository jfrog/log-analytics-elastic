# Artifactory and Xray Logging Analytics with FluentD, ElasticSearch and Kibana

The following describes how to configure Elastic and Kibana to gather metrics from Artifactory and Xray through the use of FluentD.

| version | artifactory | xray   | distribution | mission_control | pipelines |
| ------- | ----------- | ------ | ------------ | --------------- | --------- |
| 0.9.0   | 7.12.6      | 3.15.3 | 2.6.0        | 4.6.2           | 1.10.0    |
| 0.8.0   | 7.10.5      | 3.8.6  | 2.4.2        | 4.5.0           | 1.7.2     |
| 0.7.0   | 7.7.8       | 3.8.6  | 2.4.2        | 4.5.0           | 1.7.2     |
| 0.6.0   | 7.7.8       | 3.8.6  | 2.4.2        | 4.5.0           | 1.7.2     |
| 0.5.0   | 7.7.3       | 3.8.0  | 2.4.2        | 4.5.0           | 1.7.2     |
| 0.4.0   | 7.7.3       | 3.8.0  | 2.4.2        | 4.5.0           | N/A       |
| 0.3.0   | 7.7.3       | 3.8.0  | 2.4.2        | N/A             | N/A       |
| 0.2.0   | 7.7.3       | 3.8.0  | N/A          | N/A             | N/A       |
| 0.1.1   | 7.6.3       | 3.6.2  | N/A          | N/A             | N/A       |

## Environment Configuration

The environment variable JF_PRODUCT_DATA_INTERNAL must be defined to the correct location.

Helm based installs will already have this defined based upon the underlying docker images.

For non-k8s based installations below is a reference to the Docker image locations per product. Note these locations may be different based upon the installation location chosen.

```text
Artifactory:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/artifactory/
```

```text
Xray:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/xray/
```

```text
Nginx:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/nginx/
```

```text
Mision Control:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/mc/
```

```text
Distribution:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/distribution/
```

```text
Pipelines:
export JF_PRODUCT_DATA_INTERNAL=/opt/jfrog/pipelines/var/
```

## Fluentd Install

### OS / Virtual Machine

Recommended install is through fluentd's native OS based package installs:

| OS            | Package Manager | Link                                                 |
| ------------- | --------------- | ---------------------------------------------------- |
| CentOS/RHEL   | RPM (YUM)       | https://docs.fluentd.org/installation/install-by-rpm |
| Debian/Ubuntu | APT             | https://docs.fluentd.org/installation/install-by-deb |
| MacOS/Darwin  | DMG             | https://docs.fluentd.org/installation/install-by-dmg |
| Windows       | MSI             | https://docs.fluentd.org/installation/install-by-msi |

User installs can utilize the zip installer for Linux

| OS             | Package Manager | Link                                                                                                   |
| -------------- | --------------- | ------------------------------------------------------------------------------------------------------ |
| Linux (x86_64) | ZIP             | https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz |

Download it to a directory the user has permissions to write such as the `$JF_PRODUCT_DATA_INTERNAL` locations discussed above:

```text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz
```

Untar to create the folder:

```text
tar -xvf fluentd-1.11.0-linux-x86_64.tar.gz
```

Move into the new folder:

```text
cd fluentd-1.11.0-linux-x86_64
```

Run the fluentd wrapper with one argument pointed to the configuration file to load:

```text
./fluentd test.conf
```

Next steps are to setup a `fluentd.conf` file using the relevant integrations for Splunk, DataDog, Elastic, or Prometheus.

### Docker

Recommended install for Docker is to utilize the zip installer for Linux

| OS             | Package Manager | Link                                                                                                   |
| -------------- | --------------- | ------------------------------------------------------------------------------------------------------ |
| Linux (x86_64) | ZIP             | https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz |

Download it to a directory the user has permissions to write such as the `$JF_PRODUCT_DATA_INTERNAL` locations discussed above:

```text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz
```

Untar to create the folder:

```text
tar -xvf fluentd-1.11.0-linux-x86_64.tar.gz
```

Move into the new folder:

```text
cd fluentd-1.11.0-linux-x86_64
```

Run the fluentd wrapper with one argument pointed to the configuration file to load:

```text
./fluentd test.conf
```

Next steps are to setup a `fluentd.conf` file using the relevant configuration files for Elastic.

### Kubernetes

Recommended install for Kubernetes is to utilize the helm chart with the associated values.yaml in this repo.

| Product        | Example Values File             |
| -------------- | ------------------------------- |
| Artifactory    | helm/artifactory-values.yaml    |
| Artifactory HA | helm/artifactory-ha-values.yaml |
| Xray           | helm/xray-values.yaml           |

Update the values.yaml associated to the product you want to deploy with your Elastic settings.

Then deploy the helm chart such as below:

Artifactory ⎈:

```text
helm upgrade --install artifactory  jfrog/artifactory \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/artifactory-values.yaml
```

Artifactory-HA ⎈:

```text
helm upgrade --install artifactory-ha  jfrog/artifactory-ha \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/artifactory-ha-values.yaml
```

Xray ⎈:

```text
helm upgrade --install xray jfrog/xray --set xray.jfrogUrl=http://my-artifactory-nginx-url \
       --set xray.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set xray.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/xray-values.yaml
```

#### Kubernetes Deployment without Helm

To modify existing Kubernetes based deployments without using Helm users can use the zip installer for Linux:

| OS             | Package Manager | Link                                                                                                   |
| -------------- | --------------- | ------------------------------------------------------------------------------------------------------ |
| Linux (x86_64) | ZIP             | https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz |

Download it to a directory the user has permissions to write such as the `$JF_PRODUCT_DATA_INTERNAL` locations discussed above:

```text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz
```

Untar to create the folder:

```text
tar -xvf fluentd-1.11.0-linux-x86_64.tar.gz
```

Move into the new folder:

```text
cd fluentd-1.11.0-linux-x86_64
```

Run the fluentd wrapper with one argument pointed to the configuration file to load:

```text
./fluentd test.conf
```

Next steps are to setup a `fluentd.conf` file using the relevant configuration files for Elastic.

### Fluentd Configuration for Elastic

#### Download fluentd.conf

Artifactory:

```text
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.rt
```

Xray:

```text
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.xray
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/siem/elastic_siem.conf
```

Nginx:

```text
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.nginx
```

Mision Control:

```text
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.missioncontrol
```

Distribution:

```text
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.distribution
```

Pipelines:

```text
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.pipelines
```

#### Configure fluentd.conf

Integration is done by specifying the host (elasticsearch - using the above files or ip address if using other coniguration), port (9200 by default)

_index_name_ is the unique identifier based on which the index patterns can be created and filters can be applied on the log data

When _logstash_format_ option is set to true, fluentd uses conventional index name format

_type_name_ is fluentd by default and it specifies the type name to write to in the record and falls back to the default if a value is not given

_include_tag_key_ defaults to false and it will add fluentd tag in the json record if set to true

_user_ will be elastic by default

_password_ will be the password specified for elastic user in elasticsearch authentication setup

These values override the last section of the `fluentd.conf` shown below:

```
<match jfrog.**>
  @type elasticsearch
  @id elasticsearch
  host elasticsearch
  port 9200
  user "elastic"
  password <password>
  index_name unified-artifactory
  include_tag_key true
  type_name fluentd
  logstash_format false
</match>
```

Instructions to run fluentd configuration files can be found at JFrog log analytic repository's [README.](https://github.com/jfrog/log-analytics/blob/master/README.md)

Authenticate with the Artifactory API by replacing `<TOKEN>` with your bearer token in the downloaded `fluent.conf.rt` file.
There should be two spots listed below:

```
command "curl --request GET 'http://localhost:8081/artifactory/api/system/version' -H 'Authorization: Bearer <TOKEN>'"
headers {"Authorization":"Bearer <TOKEN>"}
```

For information on authentication with a bearer token with artifactory, please visit [Bearer Token Authentication](https://www.jfrog.com/confluence/display/JFROG/Access+Tokens#AccessTokens)

#### Configure splunk_siem.conf

Integration is done by setting up Xray. Obtain JPD url and access token for API. Configure the source directive parameters specified below

- **tag** (string) (required): The value is the tag assigned to the generated events.
- **jpd_url** (string) (required): JPD url required to pull Xray SIEM violations
- **access_token** (string) (required): [Access token](https://www.jfrog.com/confluence/display/JFROG/Access+Tokens) to authenticate Xray
- **pos_file** (string) (required): Position file to record last SIEM violation pulled
- **thread_count** (integer) (optional): Number of workers to process violation records in thread pool
  - Default value: `5`
- **wait_interval** (integer) (optional): Wait interval between pulling new events
  - Default value: `60`

## Elastic-Kibana Setup

Once the kibana is up, the host and port should be configured in fluent.conf and td-agent can be started. This creates an index with the name specified in the conf file

Creat an index pattern in the Management section and access the logs on the discover tab

To access already existing visualizations and filters, import [export.ndjson](https://github.com/jfrog/log-analytics/blob/master/elastic-fluentd-kibana/export.ndjson) to Saved objects in Management section

### Dashboards

- **Application** - This dashboard tracks Log Volume(information about different log sources) and Artifactory Errors over time(bursts of application errors that may otherwise go undetected)
- **Audit** - This dashboard tracks audit logs help you determine who is accessing your Artifactory instance and from where. These can help you track potentially malicious requests or processes (such as CI jobs) using expired credentials.
- **Docker** - This dashboard tracks dockerhub pull request changes (anonymous gets only 100 requests per 6 hours where as free account gets 200 requests per 6 hours)
- **Requests** - This dashboard tracks HTTP response codes, Top 10 IP addresses for uploads and downloads
- **Xray Logs** - This dashboard tracks Log Volume, Xry Log Errors over time, HTTP 500 errors and Xray HTTP Response codes
- **Xray Violations** - This dashboard gives an aggregated snapshot of all the violations, watch policies, and the total number of infected artifacts and components within their environment. This information is also organized by watch, by policy, by type and by severity to provide deeper segmentation and analysis.

## Demo Requirements

- Kubernetes Cluster
- Artifactory and/or Xray installed via [JFrog Helm Charts](https://github.com/jfrog/charts)
- Helm 3

### Installing Elasticsearch and Kibana on K8s

Elasticsearch kibana setup can be done using the following files or using manual configuration

- [Elastic_configmap](https://github.com/jfrog/log-analytics/blob/master/elastic-fluentd-kibana/elasticsearch_configmap.yaml) - Elasticsearch ConfigMap
- [Elastic_statefulset](https://github.com/jfrog/log-analytics/blob/master/elastic-fluentd-kibana/elasticsearch_statefulset.yaml) - Elasticsearch Statefulset
- [Elastic_service](https://github.com/jfrog/log-analytics/blob/master/elastic-fluentd-kibana/elasticsearch_svc.yaml) - Elasticsearch Service
- [Kibana_configmap](https://github.com/jfrog/log-analytics/blob/master/elastic-fluentd-kibana/kibana_configmap.yaml) - Kibana ConfigMap
- [Kibana_deployment](https://github.com/jfrog/log-analytics/blob/master/elastic-fluentd-kibana/kibana_deployment.yaml) - Kibana Deplpoyment
- [Kibana_service](https://github.com/jfrog/log-analytics/blob/master/elastic-fluentd-kibana/kibana_svc.yaml) - Kibana Service

### Setup

To run this integration start by creating elasticsearch configmap, service and statefulset

```
kubectl create -f elastic/elasticsearch_configmap.yaml
kubectl create -f elastic/elasticsearch_svc.yaml
kubectl create -f elastic/elasticsearch_statefulset.yaml
```

Check for the status of the statefulset using

```
kubectl rollout status sts/es-cluster
```

Setup passwords for elasticsearch using

```
kubectl exec -it $(kubectl get pods | grep es-cluster-0 | sed -n 1p | awk '{print $1}') -- bin/elasticsearch-setup-passwords interactive
```

Note the password given to elastic user and store the password in a secret

```
kubectl create secret generic elasticsearch-pw-elastic --from-literal password=<password>
```

Create Kibana configmap, service and deployment

```
kubectl create -f kibana/kibana_configmap.yaml
kubectl create -f kibana/kibana_svc.yaml
kubectl create -f kibana/kibana_deployment.yaml
```

Wait for the deployment status using

```
kubectl rollout status deployment/kibana
```

This will create a Kibana web console which can be accessed using username _elastic_ and password as specified in the interactive authentication setup

Once we have deployed elasticsearch and kibana, we can access it via kibana web console. We can check for the running logging agents in Index Management section

### Generating Data for Testing

[Partner Integration Test Framework](https://github.com/jfrog/partner-integration-tests) can be used to generate data for metrics.

## References

- [Elasticsearch](https://www.elastic.co/) - Elastic search log data platform
- [Kibana](https://www.elastic.co/kibana) - Elastic search visualization layer
