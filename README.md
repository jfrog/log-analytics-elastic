> [!WARNING]
> The integration will reach End of Support on October 31, 2023. After that date, this repository will no longer receive updates from JFrog and will no longer be supported by JFrog

# Artifactory and Xray Logging Analytics with FluentD, ElasticSearch and Kibana

The following describes how to configure Elastic and Kibana to gather metrics from Artifactory and Xray through the use of FluentD.

1. [Elastic Setup](#elastic-setup)
2. [Environment Configuration](#environment-configuration)
3. [Fluentd Installation](#fluentd-installation)
   - [OS / Virtual Machine](#os--virtual-machine)
   - [Docker](#docker)
   - [Kubernetes Deployment with Helm](#kubernetes-deployment-with-helm)
4. [Fluentd Configuration for Elastic](#fluentd-configuration-for-elastic)
5. [Dashboards](#dashboards)
6. [Splunk Demo](#splunk-demo)
7. [References](#references)

## Elastic Setup

Install the `JFrog Platform` Dashboards

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

For information on authentication with a bearer token with artifactory, please visit [Bearer Token Authentication](https://www.jfrog.com/confluence/display/JFROG/Access+Tokens#AccessTokens)

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

### Docker

In order to run fluentd as a docker image to send the log, siem and metrics data to elastic, the following commands needs to be executed on the host that runs the docker.
Change the fluentd config file based on the which component the metrics is collected.

command example

```text
docker run \
    -e JF_PRODUCT_DATA_INTERNAL='/opt/jfrog/artifactory/var/' \
    -e JF_JPD_URL='http://localhost:8082' \
    -e JF_JPD_USER_NAME='admin' \
    -e JF_JPD_API_KEY='<API_KEY>' \
    -e JF_JPD_TOKEN='<JPD_ADMIN_TOKEN>' \
    -e ELASTIC_HOST='localhost' \
    -e ELASTIC_PORT='9200' \
    -e ELASTIC_USER='elastic' \
    -e ELASTIC_PASSWORD='changeme' \
    -e ELASTIC_SCHEME='http' \
    -e ELASTIC_SSL_VERIFY='false' \
    --entrypoint '/bin/sh' partnership-public-images.jfrog.io/fluentd/fluentd:1.18.0 \
    -c 'curl https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.rt -o fluentd.conf; fluentd -v -c fluentd.conf'

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

Jfrog Platform ⎈:

Update the following fields in `/helm/jfrog-platform-values.yaml`:

Replace required parameters below:

```text
helm upgrade --install jfrog-platform jfrog/jfrog-platform \
       -f helm/jfrog-platform-values.yaml \
       --set global.elastic.host=elasticsearch-es-http.elastic.svc.cluster.local \
       --set global.elastic.port=9200 \
       --set global.elastic.user=elastic \
       --set global.elastic.password=changeme \
       --set global.elastic.scheme=https \
       --set global.elastic.ssl_verify=false \
       --set global.jfrog.observability.metrics.jpd_url=http://localhost:8082 \
       --set global.jfrog.observability.metrics.jpd_url_nginx=http://jfrog-platform-artifactory-nginx \
       --set global.jfrog.observability.metrics.username=admin \
       --set global.jfrog.observability.metrics.apikey=<API_KEY> \
       --set global.jfrog.observability.metrics.token=<JPD_ADMIN_TOKEN> \
       --set global.jfrog.observability.branch=master
```

## Fluentd Configuration for Elastic

All the dynamic configuration for fluentd is picked from environment variable for xray and artifactory.

### Fluentd configs:

- **Artifactory** - https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.rt
- **Xray** - https://raw.githubusercontent.com/jfrog/log-analytics-elastic/master/fluent.conf.xray

## Dashboards

### Artifactory dashboard

- **Application** - This dashboard tracks Log Volume(information about different log sources) and Artifactory Errors over time(bursts of application errors that may otherwise go undetected)
- **Audit** - This dashboard tracks audit logs help you determine who is accessing your Artifactory instance and from where. These can help you track potentially malicious requests or processes (such as CI jobs) using expired credentials.
- **Docker** - This dashboard tracks dockerhub pull request changes (anonymous gets only 100 requests per 6 hours where as free account gets 200 requests per 6 hours)
- **Requests** - This dashboard tracks HTTP response codes, Top 10 IP addresses for uploads and downloads
- **Xray Logs** - This dashboard tracks Log Volume, Xry Log Errors over time, HTTP 500 errors and Xray HTTP Response codes
- **Xray Violations** - This dashboard gives an aggregated snapshot of all the violations, watch policies, and the total number of infected artifacts and components within their environment. This information is also organized by watch, by policy, by type and by severity to provide deeper segmentation and analysis.

## Elastic Demo

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
