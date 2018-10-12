# boshdirector_export-release
BOSH release for boshdirector-exporter
This is a BOSH release for the bosh-director_exporter: https://github.com/orangeglasses/bosh-director_exporter
This release is intended to be a part of a BOSH deployed prometheus deployment: https://github.com/bosh-prometheus/prometheus-boshrelease

How to use this release:
```
git clone https://github.com/orangeglasses/boshdirector_export-release.git
cd boshdirector_export-release
bosh upload-release
```
# Prometheus deployment manifest changes:
use the ops file in the manifest folder in this repo (experimental) or follow the steps below.

make sure to add the release to your prometheus deployment manifest:
```
release:
- name: boshdirector_exporter-release
  version: latest
```

Add the following jobs to your prometheus instance:
```
  - name: boshdirector_exporter
    release: boshdirector_exporter-release
    properties:
      boshdirector_exporter:
        agent:
          url: ((boshdirector_agent_url))
          username: ((boshdirector_agent_username))
          password: ((boshdirector_agent_password))
          ca_cert: ((boshdirector_ca_cert))
        metrics:
          environment: ((metrics_environment))
  - name: boshdirector_alerts
    release: boshdirector_exporter-release
```

Add the following job to your grafana instance:
```
  - name: boshdirector_dashboards
    release: boshdirector_exporter-release
```

Add the following to properties.prometheus.scrape_configs in the prometheus instance group
```
        - job_name: bosh-director
          scrape_interval: 2m
          honor_labels: true
          metrics_path: '/metrics'
          params:
            'match[]':
              - '{environment="cf"}'
          static_configs:
          - targets:
            - localhost:9191
```

When deploying provide the new variables. Below is an example variable file:
```
boshdirector_agent_url: https://192.168.192.20:6868/agent 
boshdirector_agent_username: vcap
boshdirector_agent_password: P@ssw0rdP@ssw0rd
boshdirector_ca_cert: |
-----BEGIN CERTIFICATE-----
<cert here>
-----END CERTIFICATE-----
```
