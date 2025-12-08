# kubernetes-homelab

## Node Layout

talos01.mbhlab.uk - HP EliteDesk 800G1 Mini - i3 / 16gb RAM - Control Plane  
talos02.mbhlab.uk - HP EliteDesk 800G3 Mini - i5 / 32gb RAM - Worker Node  
talos03.mbhlab.uk - HP EliteDesk 800G3 Mini - i5 / 32gb RAM - Worker Node  

## Services

### Infrastructure

#### Ingress / Networking

- [X] Traefik
- [X] Cilium

#### Storage

- [X] NFS Storage Provisioner

#### Security

- [X] Cert Manager
- [X] External Secrets Operator
- [] Authentik

### Monitoring

#### Dashboards

- [X] Grafana

#### Uptime Monitoring

- [X] Gatus

#### Collectors

- [X] Kube-prometheus-stack

### Applications

- [X] Homepage
- [] Home Assistant
- [] LazyLibrarian
- [] Domain Locker
- [] MyFin Budget
- [] Immich?
- [] Version Checker

### Tips / Useful Commands

- Force update an external secret
  - ```kubectl annotate externalsecrets external-secret-name -n namespace force-sync=(date +%s) --overwrite```
