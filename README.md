# Portainer + Tailscale nodes with Prometheus (and Grafana)

This bundle gives you:
- A central **Prometheus** + **Grafana** stack you can deploy as a Portainer _Stack_ on any one node.
- A lightweight **node_exporter** stack to deploy on **each** Tailscale node you want to monitor.

## 1) Prereqs

- All nodes are on the same Tailscale tailnet and can reach each other over Tailscale.
- (Recommended) MagicDNS enabled on your tailnet so containers can resolve `*.ts.net` hostnames.
- Portainer Agent/Edge Agent installed on each node (so you can push stacks remotely).

## 2) Deploy node_exporter on every node

In Portainer, on each host:
- Stacks → **Add stack** → Upload `stack-node-exporter.yml` → **Deploy stack**.
- If you want node_exporter to bind **only** to the Tailscale IP, edit the command flag:
  `--web.listen-address=100.x.y.z:9100` and replace `100.x.y.z` with that node's Tailscale IP.
  (Otherwise it listens on all interfaces; access is still limited by your Tailscale ACLs.)

## 3) Deploy the central Prometheus + Grafana

On the node that will host Prometheus/Grafana:
- Stacks → **Add stack** → Upload `docker-compose.yml` (in this folder).
- Also upload `prometheus.yml` and the `provisioning/` folder **in the same stack context** so relative volume paths work.
- Edit `prometheus.yml`: replace the example `REPLACE_ME_NODE_A/B` targets with either MagicDNS names
  like `myhost.ts.net:9100` or literal Tailscale IPs like `100.101.102.103:9100`.
- Deploy. Prometheus: `http://<host>:9090` and Grafana: `http://<host>:3000` (admin/ChangeMeNow!).

## 4) Import a ready-made Node dashboard

In Grafana → Dashboards → Import:
- Paste **1860** ("Node Exporter Full") and select the Prometheus datasource.

## 5) Optional: Add cAdvisor for container metrics

On any host:
- Run cAdvisor (host network) and add it to `prometheus.yml` under the `cadvisor` job:
  `docker run -d --name=cadvisor --net=host --pid=host --privileged gcr.io/cadvisor/cadvisor:v0.47.2`

## 6) Tips

- If containers can't resolve `*.ts.net`, either switch to static Tailscale IPs in `prometheus.yml` **or**
  mount `/etc/resolv.conf` from the host into the Prometheus container if the host already uses MagicDNS.
- Tighten access with Tailscale ACLs so only your Prometheus host can hit nodes on port **9100**.
- Long‑term retention: move Prometheus data to a bigger disk or a remote TSDB (e.g., Thanos) later.

Enjoy! ✨
