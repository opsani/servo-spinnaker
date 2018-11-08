# servo-spinnaker
Optune adjust driver for Spinnaker

This driver updates an existing Spinnaker pipeline (provided in driver's configuration), then it calls the pipeline (either the same pipeline, or a different one) and waits for the pipeline execution to complete. Optionally, it can check if the value of an input setting matches the number of entries returned by a consul query.

Note: this driver requires `adjust.py` base class from the Optune servo core. It can be copied or symlinked here as part of packaging

# driver configuration

The following parameters can be configured for the driver. The configuration should be in a file named `config.yaml` in the same directory as the driver.

* `api_url`: Base url of the Spinnaker server
* `pipeline_user`: Spinnaker pipeline user
* `update_pipeline_name`: Name of the Spinnaker pipeline to update with the settings values
* `update_pipeline_application`: Name of the Spinnaker pipeline application to update with the settings values
* `invoke_pipeline_name`: Name of the Spinnaker pipeline to invoke (may or may not be the same as the one that is updated)
* `invoke_pipeline_application`: Name of the Spinnaker pipeline application to invoke (may or may not be the same as the one that is updated)
* `invoke_pipeline_params`:  key-value parameters pairs to pass when invoking the pipeline
* `consul_url`: (Optional) Consul URL to call in order to get list of active hosts and compare that the the value of a setting that has `use_as_required_n_hosts: True` specified. If the setting value does not match the number of hosts returned by the query, the query will be retried for up to `health_check_timeout` seconds and then the adjust operation will fail
* `health_check_timeout`: (Optional) How long (in seconds) to keep retrying the `consul_url` query and evaluation
* `api_client_cert`: (Optional) Path to client certificate to use when connecting to Spinnaker
* `api_ca_bundle`: (Optional) Path to CA file to use when connecting to Spinnaker with a self signed certificate. You can set to False to ignore self-signed certificate errors
* `consul_client_cert`: (Optional) Path to client certificate to use when connecting to Consul
* `consul_ca_bundle`: (Optional) Path to CA file to use when connecting to Consul with a self signed certificate. You can set to False to ignore self-signed certificate errors
* `components/${name}/settings`: Settings that the driver will adjust. Each setting supports the following keys:
  * `jsonpath`: list of JSON paths in the pipeline that will be updated with the value of this setting. See [jsonpath-ng docs](https://github.com/h2non/jsonpath-ng) for details on how path can be specified.
  * `use_as_required_n_hosts`: If set to `True`, the value of this setting will be used as the number of required hosts to check for when polling consul (see `consul_url`). Default: `False`

Example `config.yaml`:

```

spinnaker:
  api_url: https://api.spinnaker.com/
  pipeline_user: my-user

  update_pipeline_name: my-app
  update_pipeline_application: my-pipeline

  invoke_pipeline_name: my-app
  invoke_pipeline_application: my-pipeline
  invoke_pipeline_params:
    param1_name: param1_value
    param2_name: param2_value

  consul_url: http://example.com/foo # Optional
  health_check_timeout: 300 # Optional
  api_client_cert: path/to/file.pem # Optional
  api_ca_bundle: path/to/ca.crt # Optional
  consul_client_cert: path/to/file.pem # Optional
  consul_ca_bundle: path/to/ca.crt # Optional

  components:
    gui:
      settings:
        inst_type:
          jsonpath:
            - "$.stages[*].clusters[?(@.stack == 'my-stack')].instanceType"
        inst_count:
          use_as_required_n_hosts: True
          jsonpath:
            - "$.stages[*].clusters[?(@.stack == 'my-stack')].capacity.desired"
            - "$.stages[*].clusters[?(@.stack == 'my-stack')].capacity.min"
            - "$.stages[*].clusters[?(@.stack == 'my-stack')].capacity.max"

"""
```


