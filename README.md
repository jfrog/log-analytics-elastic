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

## Elastic Setup

Install the `JFrog Platform` integration

```text
1. In Kibana, go to Management > Integrations > Browse integrations. Search for "JFrog Platform" and select an integration
2. Click the Settings tab.
3. Click Install <integration> assets to set up the Kibana and Elasticsearch assets.
```

Installing all kibana assests directly instead of integration.
To access already existing visualizations and filters, import [kibana_dashboard_v1.ndjson](https://github.com/jfrog/log-analytics-elastic/blob/master/kibana/kibana_dashboard_v1.ndjson) to Saved objects in Management section

`Note! Only elastic verion 7.14 and above is supported`

## Environment Configuration

We rely heavily on environment variables so that the correct log files are streamed to your observability dashboards. Ensure that you set the JF_PRODUCT_DATA_INTERNAL environment variable to the correct path for your product

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
Mission Control:
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

```text
Common:
export JF_JPD_URL=http://localhost:8082
export JF_JPD_USER_NAME=admin
export JF_JPD_API_KEY=<API_KEY>
export JF_JPD_TOKEN=<JPD_ADMIN_TOKEN>
export ELASTIC_HOST=elasticsearch-es-http.elastic.svc.cluster.local
export ELASTIC_PORT=9200
export ELASTIC_USER=elastic
export ELASTIC_PASSWORD=changeme
export ELASTIC_SCHEME=https
export ELASTIC_SSL_VERIFY=true
```

## Fluentd Installation

### OS / Virtual Machine

Ensure you have access to the Internet from VM. Recommended install is through fluentd's native OS based package installs:

| OS              | Package Manager     | Link                                                 |
| --------------- | ------------------- | ---------------------------------------------------- |
| CentOS/RHEL     | Linux - RPM (YUM)   | https://docs.fluentd.org/installation/install-by-rpm |
| Debian/Ubuntu   | Linux - APT         | https://docs.fluentd.org/installation/install-by-deb |
| MacOS/Darwin    | MacOS - DMG         | https://docs.fluentd.org/installation/install-by-dmg |
| Windows         | Windows - MSI       | https://docs.fluentd.org/installation/install-by-msi |
| Gem Install\*\* | MacOS & Linux - Gem | https://docs.fluentd.org/installation/install-by-gem |

```text
** For Gem based install, Ruby Interpreter has to be setup first, following is the recommended process to install Ruby

1. Install Ruby Version Manager (RVM) as described in https://rvm.io/rvm/install#installation-explained, ensure to follow all the onscreen instructions provided to complete the rvm installation
	* For installation across users a SUDO based install is recommended, the installation is as described in https://rvm.io/support/troubleshooting#sudo

2. Once rvm installation is complete, verify the RVM installation executing the command 'rvm -v'

3. Now install ruby v2.7.0 or above executing the command 'rvm install <ver_num>', ex: 'rvm install 2.7.5'

4. Verify the ruby installation, execute 'ruby -v', gem installation 'gem -v' and 'bundler -v' to ensure all the components are intact

5. Post completion of Ruby, Gems installation, the environment is ready to further install new gems, execute the following gem install commands one after other to setup the needed ecosystem

	'gem install fluentd'

```

After FluentD is successfully installed, the below plugins are required to be installed

```text

	'gem install fluent-plugin-elasticsearch'
	'gem install fluent-plugin-jfrog-siem'
	'gem install fluent-plugin-jfrog-metrics'

```

Configure `fluent.conf.*` according to the instructions mentioned in [Fluentd Configuration for Elastic](#fluentd-configuration-for-elastic) section and then run the fluentd wrapper with one argument pointed to the `fluent.conf.*` file configured.

```text
./fluentd $JF_PRODUCT_DATA_INTERNAL/fluent.conf.<product_name>
```

### Kubernetes Deployment with Helm

Recommended installation for Kubernetes is to utilize the helm chart with the associated values.yaml in this repo.

| Product        | Example Values File             |
| -------------- | ------------------------------- |
| Artifactory    | helm/artifactory-values.yaml    |
| Artifactory HA | helm/artifactory-ha-values.yaml |
| Xray           | helm/xray-values.yaml           |
| JFrog Platform | helm/jfrog-platform-values.yaml |

Update the values.yaml associated to the product you want to deploy with your elastic settings.

Then deploy the helm chart as described below:

Add JFrog Helm repository:

```text
helm repo add jfrog https://charts.jfrog.io
helm repo update
```

Replace placeholders with your `masterKey` and `joinKey`. To generate each of them, use the command
`openssl rand -hex 32`

Artifactory ⎈:

Replace the required parameters below:

```text
helm upgrade --install artifactory  jfrog/artifactory \
       -f helm/artifactory-values.yaml \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       --set global.elastic.host=<ELASTIC_CLUSTER_URL> \
       --set global.elastic.port=9200 \
       --set global.elastic.user=elastic \
       --set global.elastic.password=password \
       --set global.elastic.scheme=https \
       --set global.elastic.ssl_verify=true \
       --set global.jfrog.observability.metrics.jpd_url=http://localhost:8082 \
       --set global.jfrog.observability.metrics.jpd_url_nginx=http://jfrog-platform-artifactory-nginx \
       --set global.jfrog.observability.metrics.username=admin \
       --set global.jfrog.observability.metrics.apikey=<API_KEY> \
       --set global.jfrog.observability.metrics.token=<JPD_ADMIN_TOKEN> \
       --set global.jfrog.observability.branch=master
```

Artifactory-HA ⎈:

For HA installation, please create a license secret on your cluster prior to installation.

```text
kubectl create secret generic artifactory-license --from-file=<path_to_license_file>artifactory.cluster.license
```

Note: Replace placeholders with your `masterKey` and `joinKey`. To generate each of them, use the command
`openssl rand -hex 32`

Replace required values below:

```text
helm upgrade --install artifactory-ha  jfrog/artifactory-ha \
       -f helm/artifactory-ha-values.yaml
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       --set global.elastic.host=<ELASTIC_CLUSTER_URL> \
       --set global.elastic.port=9200 \
       --set global.elastic.user=elastic \
       --set global.elastic.password=password \
       --set global.elastic.scheme=https \
       --set global.elastic.ssl_verify=true \
       --set global.jfrog.observability.metrics.jpd_url=http://localhost:8082 \
       --set global.jfrog.observability.metrics.jpd_url_nginx=http://jfrog-platform-artifactory-nginx \
       --set global.jfrog.observability.metrics.username=admin \
       --set global.jfrog.observability.metrics.apikey=<API_KEY> \
       --set global.jfrog.observability.metrics.token=<JPD_ADMIN_TOKEN> \
       --set global.jfrog.observability.branch=master
```

Xray ⎈:

Update the following fields in `/helm/xray-values.yaml`:

Use the same `joinKey` as you used in Artifactory installation to allow Xray node to successfully connect to Artifactory.

Replace required parameters below:

```text
helm upgrade --install xray jfrog/xray \
       -f helm/xray-values.yaml
       --set xray.jfrogUrl=http://my-artifactory-nginx-url \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       --set global.elastic.host=<ELASTIC_CLUSTER_URL> \
       --set global.elastic.port=9200 \
       --set global.elastic.user=elastic \
       --set global.elastic.password=password \
       --set global.elastic.scheme=https \
       --set global.elastic.ssl_verify=true \
       --set global.jfrog.observability.metrics.jpd_url=http://localhost:8082 \
       --set global.jfrog.observability.metrics.jpd_url_nginx=http://my-artifactory-nginx-url \
       --set global.jfrog.observability.metrics.username=admin \
       --set global.jfrog.observability.metrics.apikey=<API_KEY> \
       --set global.jfrog.observability.metrics.token=<JPD_ADMIN_TOKEN> \
       --set global.jfrog.observability.branch=master
```

### Configuration steps for Nginx

Download the Nginx fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

```text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.nginx
```

Override the match directive(last section) of the downloaded `fluent.conf.nginx` with the details given below

```
<match jfrog.**>
  @type elasticsearch
  @id elasticsearch
  host my_host
  port 9200
  user my_user
  password my_password
  index_name unified-nginx
  include_tag_key true
  type_name fluentd
  logstash_format false
</match>
```

## Dashboards

### Artifactory dashboard

- **Application** - This dashboard tracks Log Volume(information about different log sources) and Artifactory Errors over time(bursts of application errors that may otherwise go undetected)
- **Audit** - This dashboard tracks audit logs help you determine who is accessing your Artifactory instance and from where. These can help you track potentially malicious requests or processes (such as CI jobs) using expired credentials.
- **Docker** - This dashboard tracks dockerhub pull request changes (anonymous gets only 100 requests per 6 hours where as free account gets 200 requests per 6 hours)
- **Requests** - This dashboard tracks HTTP response codes, Top 10 IP addresses for uploads and downloads
- **Xray Logs** - This dashboard tracks Log Volume, Xry Log Errors over time, HTTP 500 errors and Xray HTTP Response codes
- **Xray Violations** - This dashboard gives an aggregated snapshot of all the violations, watch policies, and the total number of infected artifacts and components within their environment. This information is also organized by watch, by policy, by type and by severity to provide deeper segmentation and analysis.

# Demo Requirements

- Kubernetes Cluster
- Artifactory and/or Xray installed via [JFrog Helm Charts](https://github.com/jfrog/charts)
- Helm 3

### Installing Elasticsearch and Kibana on K8s

Elasticsearch kibana setup can be done using the following files or using manual configuration

Install ECK operator following https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html

- [Elasticserach Cluster](https://github.com/jfrog/log-analytics-elastic/blob/master/elastic/k8s-operator/elasticsearch_cluster.yaml) - Elasticsearch Cluster
- [Kibana](https://github.com/jfrog/log-analytics-elastic/blob/master/elastic/k8s-operator/kibana.yaml) - Kibana

### Setup

To run this integration start by creating elasticsearch and kibana

```
kubectl create -f elastic/k8s-operator/elasticsearch_cluster.yaml
kubectl create -f elastic/k8s-operator/kibana.yaml
```

## References

- [Elasticsearch](https://www.elastic.co/) - Elastic search log data platform
- [Kibana](https://www.elastic.co/kibana) - Elastic search visualization layer
