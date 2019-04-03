# Keycloak Gatekeeper

Keycloak Gatekeeper is a forward and reverse HTTP(S) proxy to put in front of web applications and services where it is not possible to install the Keycloak adapter. You can set up URL filters so that certain URLs are secured either by browser login and/or bearer token authentication. You can also define role constraints for URL patterns within your applications.

## TL;DR;

```console
$ helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
$ helm install incubator/keycloak-gatekeeper
```

## Introduction

This chart bootstraps a [Keycloak Proxy](https://www.keycloak.org/docs/4.6/securing_apps/index.html#_keycloak_generic_adapter) Deployment on a [Kubernetes](https://kubernetes.io) cluster
using the [Helm](https://helm.sh) package manager. It provisions a fully featured Keycloak Proxy installation.
For more information on Keycloak and its capabilities, see its [documentation](https://www.keycloak.org/docs/4.6/securing_apps/index.html#_keycloak_generic_adaptery) and [Docker Hub repository](https://hub.docker.com/r/jboss/keycloak-gatekeeper/).

## Prerequisites Details

Keycloak Proxy is designed primarily for Keycloak, an OpenID Connect identity provider. But it also works with other OpenID Connect identity providers.

## Installing the Chart

To install the chart with the release name `keycloak-gatekeeper`:

```console
$ helm install --name keycloak-gatekeeper incubator/keycloak-gatekeeper
```

## Uninstalling the Chart

To uninstall/delete the `keycloak-gatekeeper` deployment:

```console
$ helm delete keycloak-gatekeeper
```

## Configuration

The following table lists the configurable parameters of the Keycloak chart and their default values.

The full list of parameters are available in the [values.yaml](./values.yaml).

Parameter | Description | Default
--- | --- | ---
`image.repository` | Keycloak Proxy image repository | `jboss/keycloak-gatekeeper`
`image.tag` | Keycloak Proxy image version | `5.0.0`
`image.pullPolicy` | Keycloak Proxy image pull policy | `IfNotPresent`
`service.type` | The service type | `ClusterIP`
`service.port` | The service port | `80`
`service.nodePort` | The service nodePort | `""`
`ingress.enabled` | If true, an ingress is be created | `false`
`ingress.annotations` | Annotations for the ingress | `{}`
`ingress.path` | Path for backend | `/`
`ingress.hosts` | A list of hosts for the ingresss | `[keycloak-gatekeeper.example.com]`
`ingress.tls.secretName` | If tls is enabled, uses an existing secret with this name | `""`
`ingress.tls.hosts` | A list of hosts for   | `""`
`resources` | CPU/Memory resource requests/limits | `{}`
`nodeSelector` | Node labels for pod assignment | `{}`
`tolerations` | Tolerations for pod assignment | `[]`
`affinity` | Node/Pod affinities | `{}`
`configmap.listen` | the interface the service should be listening on [$PROXY_LISTEN], all interfaces is specified as ':<port>', unix sockets as unix://<REL_PATH> or </ABS PATH> | 0.0.0.0:3000 
`configmap.discoveryUrl` | **Mandatory**: discovery url to retrieve the openid configuration [$PROXY_DISCOVERY_URL] | https://keycloack.example.com/auth/realms/default
`configmap.clientId` | **Mandatory**: client id used to authenticate to the oauth service [$PROXY_CLIENT_ID] | CLIENT_ID
`configmap.clientSecret` | **Mandatory**: client secret used to authenticate to the oauth service [$PROXY_CLIENT_SECRET] | CLIENT_SECRET
`configmap.redirectionUrl` | **Mandatory**: redirection url for the oauth callback url, defaults to host header is absent | https://mysite.example.com
`configmap.upstreamUrl` | **Mandatory**:  url for the upstream endpoint you wish to proxy [$PROXY_UPSTREAM_URL] | http://servicename.namespace.svc.cluster.local`
`configmap.resources` | a collection of resource i.e. urls that you wish to protect | 'uri:\n /*'

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

For array of strings, specify each parameter in separated `--set` arguments.

```yaml
resources:
  - uri: /admin*
    methods:
      - GET
    roles:
      - client:test1
      - client:test2
    require-any-role: true
    groups:
      - admins
      - users
  - uri: /backend*
    roles:
    - client:test1
  - uri: /public/*
    white-listed: true
  - uri: /favicon
    white-listed: true
  - uri: /css/*
    white-listed: true
  - uri: /img/*
    white-listed: true
```

The above complex YAML structure results in the following Helm arguments.

```bash
--set configmap.resources[0].uri="/admin*" \
--set configmap.resources[0].methods[0]=GET \
--set configmap.resources[0].roles[0]="client:test1" \
--set configmap.resources[0].roles[1]="client:test2" \
--set configmap.resources[0].require-any-role="true" \
--set configmap.resources[0].groups[0]="admins" \
--set configmap.resources[0].groups[1]="users" \
--set configmap.resources[1].uri="/backend*" \
--set configmap.resources[1].roles[0]="client:test1" \
--set configmap.resources[2].uri="/public/*" \
--set configmap.resources[2].white-listed="true" \
--set configmap.resources[3].uri="/favicon" \
--set configmap.resources[3].white-listed="true" \
--set configmap.resources[4].uri="/css/*" \
--set configmap.resources[4].white-listed="true" \
--set configmap.resources[5].uri="/img/*" \
--set configmap.resources[5].white-listed="true"
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install --name keycloak-gatekeeper -f values.yaml incubator/keycloak-gatekeeper
```

## Proxy Configuration

The following configurations which are located in a configmap are required to request authentication and authorization.
Please refer to [Keycloak Proxy](https://www.keycloak.org/docs/4.6/securing_apps/index.html#_keycloak_generic_adapter) and [Adapter Config](https://www.keycloak.org/docs/4.6/securing_apps/index.html#_java_adapter_config) for more information.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-keycloak-gatekeeper
  labels:
    app: keycloak-gatekeeper
    chart: keycloak-gatekeeper-0.0.1
    release: RELEASE-NAME
    heritage: Tiller
data:
  config.yml: |+
    listen: 0.0.0.0:3000
    discovery-url: https://keycloak.example.com/auth/realms/default
    client-id: CLIENT_ID
    client-secret: CLIENT_SECRET
    redirection-url: http://mysite.example.com
    skip-openid-provider-tls-verify: false
    openid-provider-timeout: 30s
    upstream-url: http://servicename.namespace.svc.cluster.local
    resources:
      - uri: /admin*
        methods:
          - GET
        roles:
          - client:test1
          - client:test2
        groups:
          - admins
          - users
        require-any-role: true
      - uri: /backend*
        roles:
          - client:test1
      - uri: /public/*
        white-listed: true
      - uri: /favicon
        white-listed: true
      - uri: /css/*
        white-listed: true
      - uri: /img/*
        white-listed: true
    preserve-host: false
    request-id-header: X-Request-ID
    enable-self-signed-tls: false
    self-signed-tls-expiration: 3h0m0s
    enable-request-id: false
    enable-logout-redirect: false
    enable-default-deny: true
    enable-encrypted-token: false
    enable-session-cookies: true
    enable-logging: true
    enable-json-logging: true
    enable-forwarding: false
    enable-security-filter: false
    enable-refresh-tokens: true
    enable-login-handler: false
    enable-token-header: true
    enable-authorization-header: true
    enable-authorization-cookies: false
    enable-https-redirection: false
    enable-profiling: false
    enable-metrics: true
    filter-browser-xss: false
    filter-content-nosniff: false
    filter-frame-deny: false
    localhost-metrics: false
    access-token-duration: 720h0m0s
    cookie-access-name: kc-access
    cookie-refresh-name: kc-state
    secure-cookie: true
    http-only-cookie: true
    skip-upstream-tls-verify: true
    cors-credentials: false
    cors-max-age: 0s
    encryption-key: "jOljNzMIBv4yynVIkFjbMljPLytCRXW3"
    invalid-auth-redirects-with-303: false
    no-redirects: false
    skip-token-verification: false
    upstream-keepalives: true
    upstream-timeout: 10s
    upstream-keepalive-timeout: 10s
    upstream-tls-handshake-timeout: 10s
    upstream-response-header-timeout: 10s
    upstream-expect-continue-timeout: 10s
    enabled-proxy-protocol: false
    max-idle-connections: 0
    max-idle-connections-per-host:
    server-read-timeout: 10s
    server-write-timeout: 10s
    server-idle-timeout: 2m0s
    use-letsencrypt: false
    letsencrypt-cache-dir: ./cache/
    disable-all-logging: false
```
## Demo

[Keycloak Proxy Demo](https://github.com/YunSangJun/keycloak-gatekeeper-demo) will help you understand the concept and behavior of this Proxy.
