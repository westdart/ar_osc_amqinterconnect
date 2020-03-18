# amq_ic_config

Generate the config files required to build out amq_interconnect / amq broker nodes

## Requirements
- AMQ Interconnect images are available (ref: 'amqic_image')

## Role Variables

| Variable                               | Description                                                                      | Default |
| --------                               | -----------                                                                      | ------- |
| ar_osc_amqinterconnect_ns              | The kubernetes namespace on which to work (will be created if does not exist)    | None    |
| ar_osc_amqinterconnect_name            | The name of the AMQ Interconnect instance                                        | None    |
| ar_osc_amqinterconnect_instance        | Object defining the AMQ IC instance (see below)                                  | None    |
| ar_osc_amqinterconnect_ic_username     | The SASL username used when connecting to upstream instances                     | None    |
| ar_osc_amqinterconnect_ic_password     | The SASL password used when connecting to upstream instances                     | None    |
| ar_osc_amqinterconnect_broker_username | The AMQ Broker username used when connecting to downstream messaging brokers     | None    |
| ar_osc_amqinterconnect_broker_password | The AMQ Broker password used when connecting to downstream messaging brokers     | None    |
| ar_osc_amqinterconnect_tls_key         | The TLS key used to establish TLS connections with upstream instances            | None    |
| ar_osc_amqinterconnect_cert_content    | The content of the signed certificate created off ar_osc_amqinterconnect_tls_key | None    |
| ar_osc_amqinterconnect_ca_cert_content | The content of the certificate trust chain                                       | None    |

The 'ar_osc_amqinterconnect_instance' variable is an object that contains the details on each instance required.

The structure is:
```
  {
    name: "<the name>",
    amqic_image: <the image>, # Optional, can be defined globally
    internal_host: "<amq interconnect service endpoint>",
    external_host: "<amq interconnect route endpoint>",
    wayPointPatterns: [<list of wapoint patterns handled by this instance>],
    incomingAddressList: [<List of incomming addresses handled by this instance>],
    oc_login_url: "<The Openshift management endpoint URL>"
  }
```

## Default Variables
| Variable                                   | Description                                             | Default                                                                                                                                               |
| --------                                   | -----------                                             | -------                                                                                                                                               |
| ar_osc_amqinterconnect_image               | The full name for the docker image to use               | 'amqic_image' taken from the 'ar_osc_amqinterconnect_instance' or global ansible variable namespace                                                   |
| ar_osc_amqinterconnect_cert_mountpoint     | Directory where the TLS certificates are placed         | '/etc/qpid-dispatch-certs'                                                                                                                            |
| ar_osc_amqinterconnect_common_name         | Application name                                        | 'name' taken from the 'ar_osc_amqinterconnect_instance' and converted to lowercase                                                                    |
| ar_osc_amqinterconnect_k8s_template        | Reference to the kubernetes application template to use | 'amqic_template' taken either from the 'ar_osc_amqinterconnect_instance' or global ansible variable namespace, otherwise 'amq-interconnect-basic.yml' |
| ar_osc_amqinterconnect_config_dest         | Directory where application artifacts are generated     | '/tmp'                                                                                                                                                |
| ar_osc_amqinterconnect_openshift_login_url | The Openshift cluster to connect to                                        | 'oc_login_url' taken either from the 'ar_osc_amqbroker_instance' or global ansible variable namespace                                                                                   |

### Globals
The following are variable used within the role:

| Variable       | Description                                                                                | Default |
| --------       | -----------                                                                                | ------- |
| amqic_template | Reference to the application template to use (must exist in either files or templates dir) | None    |
| amqic_image    | Full docker reference to the image required                                                | None    |
| oc_login_url   | The Openshift management endpoint URL (Can also be provided as an instance variable)       | None    |


## Dependencies
- ar_os_common
- ar_os_seed
- casl-ansible/roles/openshift-labels

## Example Playbook

```
- name: Setup AMQ Interconnect
  hosts: localhost

    - name: "Include amq_ic_config role for {{ ar_osc_amqinterconnect_instance.name }}"
      include_role:
        name: amq_ic_config
      vars:
        ar_osc_amqinterconnect_image:           "docker-registry-default.t2.training.local/imgns/amq-interconnect:1.3"
        ar_osc_amqinterconnect_config_dest:     "{{ generated_config_path }}/{{ ar_osc_amqinterconnect_instance.name }}"
        ar_osc_amqinterconnect_ca_cert_content: "{{ lookup('file', local_cert_path + '/' + ca_name + '.crt') }}"
        ar_osc_amqinterconnect_cert_content:    "{{ lookup('file', local_cert_path + '/' + ar_osc_amqinterconnect_instance.name + '.crt') }}"
        ar_osc_amqinterconnect_instance: {
          name: "MESH",
          internal_host: "amq-interconnect.dev-mesh.svc",
          external_host: "amq-interconnect-dev-mesh.a1.training.local",
          wayPointPatterns: ['mod.logistics.mjdi'],
          incomingAddressList: ['mod.logistics.mjdi.MESH']
        }
```

## License

MIT / BSD

## Author Information

This role was created in 2020 by David Stewart (dstewart@redhat.com)
