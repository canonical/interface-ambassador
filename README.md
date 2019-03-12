Overview
========

This repository hosts code for the Ambassador interface. It currently
allows detecting the presence of Ambassador via relations. A Kubernetes
charm can use the presence of this relation to know that it should
annotate itself with information about which routes it should be
reverse-proxied to.

Usage
=====

```yaml
# metadata.yaml
...
provides:
  ambassador:
    interface: ambassador
```

```yaml
# layer.yaml
includes:
  ...
  - "interface:ambassador"
```

```python
@when('endpoint.ambassador.joined')
def ambassador_available(_):
    config = hookenv.config()
    config['ambassador-available'] = True
    config.save()
    clear_flag('charm.name.started')


@when_not('endpoint.ambassador.joined')
def ambassador_unavailable():
    config = hookenv.config()
    config['ambassador-available'] = False
    config.save()
    clear_flag('charm.name.started')

@when_not('charm.name.started')
def start_charm():
    if config.get('ambassador-available'):
        annotations = {
            'getambassador.io/config': yaml.dump([
                {
                    'apiVersion': 'ambassador/v0',
                    'kind':  'Mapping',
                    'name':  'example_route',
                    'prefix': '/route/',
                    'rewrite': '/route/',
                    'service': f'{hookenv.service_name()}:8080',
                },
            ]),
        }
    else:
        annotations = {}

    ...

    layer.caas_base.pod_spec_set({
        'service': {'annotations': annotations},
        ...
    })
```
