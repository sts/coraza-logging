<h1>
   <img src="https://coraza.io/images/logo_shield_only.png" align="left" height="46px" alt=""/>&nbsp;
   <span>Coraza Grafana Log Pipeline Poc</span>
</h1>


## Grafana Dashboard
This repo contains a Grafana dashboard which can be used when setting up Coraza WAF logging with Loki.

_________________________


## Grafana Cloud Pipeline POC

Grafana provides free cloud-hosted instances, this guide sets up a proof-of-concept for a Coraza WAF > Grafana Cloud Loki log shipper, which should be up-and-running within the hour.

### Free Cloud Account

Sign up for a [free Grafana Cloud account](https://grafana.com/auth/sign-up/create-user), which includes a cloud-hosted Grafana instance and 50 GB of Loki log storage.

### Cloud API Key

Retrieve an new Grafana Cloud API key for shipping logs via fluent-bit to the hosted Loki instance from [Grafana Cloud Account Home](https://grafana.com/profile/org/api-keys?src=hg_home). *+ Add API Key > Name: corazawaf, Role: MetricsPusher*

### Install fluent-bit
Install fluent-bit as a log shipper:

```
curl https://packages.fluentbit.io/fluentbit.key | gpg --dearmor > /usr/share/keyrings/fluentbit-keyring.gpg

# Add the fluent-bit repository
cat <<EOF > /etc/apt/sources.list.d/fluent-bit.list
deb [signed-by=/usr/share/keyrings/fluentbit-keyring.gpg] https://packages.fluentbit.io/debian/bullseye bullseye main
EOF

apt update && apt install fluent-bit
```

Use the fluent-bit configuration from [fluent-bit/fluent-bit.conf](fluent-bit/fluent-bit.conf) as a starter, add your Grafana Cloud API key.

### Alternate audit.log format

Grafana and Loki currently have a limited set of capabilities to filter JSON objects, so you will have to ship a separate stream of the `messages: []` array of Corazas log format.

First enable the audit log in coraza.conf:

```
SecAuditLog /var/log/coraza-spoa/audit.log
SecAuditLogFormat json
SecAuditLogType Serial
```

Run the following command in a tmux session:

```
tail -1f /var/log/coraza-spoa/audit.log  | jq --unbuffered -c '.transaction.id as $id | .messages[] | { "transaction_id": $id } + .' >> /var/log/coraza-spoa/audit_messages.log
```

### Import the Grafana Dashboard

In your Grafana Cloud instance open: *Dashboards > Browse > Import* and paste the JSON contained in [grafana/dashboard.json](grafana/dashboard.json).
