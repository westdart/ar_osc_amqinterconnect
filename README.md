# amq_ic_config

Generate the config files required to build out amq_interconnect / amq broker nodes

## Requirements


## Role Variables

| Variable                           | Description                                           | Default |
| --------                           | -----------                                           | ------- |
| ar_osc_amqinterconnect_instance    | Object defining the AMQ IC instance (see below)       | None    |
| ar_osc_amqinterconnect_config_dest | Path to where generated config files are placed       | '/tmp'  |
| deployment_phase                   | Label denoting the phase (DEV, TEST etc)              | None    |
| CA_CONTENT                         | The content of the CA Certificate                     | None    |
| CERT_CONTENT                       | The content of the Application Certificate            | None    |
| QD_ROUTER_CONFIG                   | The content of the QPid Dispatch router configuration | None    |


The 'ar_osc_amqinterconnect_instance' variable is an object that contains the details on each instance required.

The structure is:
```
  {
    name: "<the name>",
    internal_host: "<amq interconnect service endpoint>",
    external_host: "<amq interconnect route endpoint>",
    wayPointPatterns: [<list of wapoint patterns handled by this instance>],
    incomingAddressList: [<List of incomming addresses handled by this instance>],
  }
```


## Default Variables

| Variable                                | Description                          | Default                                 |                                                  |          |
| --------                                | -----------                          | -------                                 |                                                  |          |
| ar_osc_amqinterconnect_cert_mountpoint: | Where the cert files will be mounted | /etc/qpid-dispatch-certs                |                                                  |          |
| ar_osc_amqinterconnect_ns:              | Openshift Namespace / Project        | {{ deployment_phase                     | lower }}-{{ ar_osc_amqinterconnect_instance.name | lower }} |
| ar_osc_amqinterconnect_common_name:     | Application name                     | {{ ar_osc_amqinterconnect_instance.name | lower }}                                         |          |



## Dependencies

- openshift-applier


## Example Playbook

```
- name: Setup AMQ Interconnect
  hosts: localhost

    - name: "Include amq_ic_config role for {{ ar_osc_amqinterconnect_instance.name }}"
      include_role:
        name: amq_ic_config
      vars:
        ar_osc_amqinterconnect_ca_cert: "{{ ca_name }}.crt"
        ar_osc_amqinterconnect_config_dest:    "{{ generated_config_path }}/{{ ar_osc_amqinterconnect_instance.name }}"
        CA_CONTENT:            "{{ lookup('file', local_cert_path + '/' + ca_name + '.crt') }}"
        CERT_CONTENT:          "{{ lookup('file', local_cert_path + '/' + ar_osc_amqinterconnect_instance.name + '.crt') }}"
        QD_ROUTER_CONFIG:      "{{ lookup('file', ar_osc_amqinterconnect_config_dest +'/' + 'qpid-dispatch-router.conf') }}"
        ar_osc_amqinterconnect_instance: {
          name: "MESH",
          internal_host: "amq-interconnect.dev-mesh.svc",
          external_host: "amq-interconnect-dev-mesh.a1.training.local",
          wayPointPatterns: ['mod.logistics.mjdi'],
          incomingAddressList: ['mod.logistics.mjdi.MESH']
        }
```

## License

BSD

## Author Information

dstewart@redhat.com
