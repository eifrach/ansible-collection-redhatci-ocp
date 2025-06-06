# utils

This is a helper role that is meant to be consumed by acm related roles (acm_sno, acm_hypershift, etc).
Brings functionality that is commonly used among those roles.

## Variables

| Variable                            | Default                             | Required by                      | Description
| ----------------------------------- | ----------------------------------- | -------------------------------- | -----------
| utils_cluster_name                  | None                                | get-credentials, monitor-install | Name of the spoke cluster
| utils_cluster_namespace             | None                                | get-credentials, monitor-install | Namespace for the spoke cluster
| utils_monitor_timeout               | 90                                  | monitor-install                  | Timeout in minutes for the installation process.
| utils_monitor_wait_time             | 3                                   | monitor-install                  | Wait time in minutes between each progress check.
| utils_imagesource_file              | None                                | image-sources                    | File with Image Digest Mirror Sets or Image Content Source Policies to transform into registries.conf format
| utils_hub_mirrors                   | None                                | image-sources                    | List of mirrors and sources in the Hub cluster.
| utils_registry                      | None                                | image-sources                    | The custom registry to use for the registries.conf.
| utils_unqualified_search_registries | ['registry.redhat.io', 'docker.io'] | image-sources                    | List of unqualified search registries.
| utils_registries                    | None                                | disconnect-agent                 | The `registries.conf` content for Assisted Images service, generated by the `image-sources` module.
| utils_ca_bundle                     | None                                | disconnect-agent                 | CA bundle for the custom registry to use in the Assisted Images service.
| utils_cm_name                       | mirror-registry                     | disconnect-agent                 | Name of the ConfigMap to include the registries and CA bundle.
| utils_ocp_version                   | None                                | disconnect-agent                 | OpenShift version to use by the Assisted Images service.
| utils_iso_url                       | None                                | disconnect-agent                 | The URL to the ISO image to use in the Assisted Images service.
| utils_root_fs_url                   | None                                | disconnect-agent                 | The URL to the rootfs image to use in the Assisted Images service.

## Utilities

### Get credentials

Gets kubeadmin password and kubeconfig from the hub cluster for the managed/spoke cluster and generates 3 variables with that information

- acm_kubeconfig_text
- acm_kubeconfig_user
- acm_kubeconfig_pass

### Monitor install

Monitors the installation of a spoke cluster through the AgentClusterInstall.

### Image sources

Transforms and combines an ImageDigestMirrorSet or ImageContentSourcePolicy file with the Hub mirrors and sources
into the `registries.conf` format using a custom registry.

### Disconnect agent

Disconnects the Assisted Service Agent by using a ConfigMap with the `registries.conf` content and CA bundle, as
well as information about the OpenShift version and the ISO and rootfs images to use.

## Usage example

See below some examples of how to use the redhatci.ocp.acm.utils role 

### Example: Get credentials

Get credentials from a deployed cluster through ACM

```yaml
- name: Get credentials from deployed cluster
  vars:
    utils_cluster_name: spoke-cluster
    utils_cluster_namespace: my-ns
  ansible.builtin.include_role:
    name: redhatci.ocp.acm.utils
    tasks_from: get-credentials
```

### Example: Monitor install

Monitor installation of a cluster through ACM

```yaml
- name: Monitor install of cluster
  vars:
    utils_cluster_name: spoke-cluster
    utils_cluster_namespace: my-ns
    utils_monitor_timeout: 60
    utils_monitor_wait_time: 20
  ansible.builtin.include_role:
    name: redhatci.ocp.acm.utils
    tasks_from: monitor-install
```

### Example: Image Sources

Transform and combine an ImageDigestMirrorSet with given hub mirrors and sources into a `registries.conf`
format using a custom registry.

This task generates the `utils_acm_registries` variable containing the transformed and combined content.

```yaml
- name: Transform ImageDigestMirrorSet
  vars:
    utils_hub_mirrors:
      - mirrors:
        - my.local.registry/path/to/image
        source: quay.io/path/to/image
      - mirrors:
        - my.local.registry/path/to/another/image
        source: quay.io/path/to/another/image
    utils_imagesource_file: /path/to/idms.yaml
    utils_registry: "my.local.registry"
    utils_unqualified_search_registries:
      - quay.io
      - my.local.registry
  ansible.builtin.include_role:
    name: redhatci.ocp.acm.utils
    tasks_from: image-sources
```

### Example: Disconnect Agent

```yaml
- name: Disconnect agent
  vars:
    utils_registries: "{{ utils_acm_registries }}"
    utils_ca_bundle: "{{ content_of_ca_bundle }}"
    utils_cm_name: my-mirror-registry
    utils_ocp_version: 4.18.19
    utils_iso_url: https://webserver.local/path/to/rhcos-418.94.202501221327-0-live.x86_64.iso
    utils_root_fs_url: https://webserver.local/path/to/rhcos-418.94.202501221327-0-live-rootfs.x86_64.img
  ansible.builtin.include_role:
    name: redhatci.ocp.acm.utils
    tasks_from: disconnect-agent
```