Matrix Synapse Server

```
sudo docker run -it --rm \
    -v "/var/matrix:/data" \
    -e SYNAPSE_SERVER_NAME=matrix.cylindric.net \
    -e SYNAPSE_REPORT_STATS=yes \
    matrixdotorg/synapse:latest generate
```

```
allow_public_rooms_without_auth: false
allow_public_rooms_over_federation: false
```

```
sudo docker run -d --name synapse \
    -v "/var/matrix:/data" \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```

```
sudo docker exec -it synapse register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml --help

sudo docker exec -it synapse register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml -a
sudo docker exec -it synapse register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml
```

```
echo [Synapse] > /etc/ufw/applications.d/synapse
echo title=Synapse >> /etc/ufw/applications.d/synapse
echo description=Matrix Synapse. >> /etc/ufw/applications.d/synapse
echo ports=8008,8448/tcp >> /etc/ufw/applications.d/synapse

ufw app update synapse
ufw allow synapse
```

```Caddyfile
matrix.cylindric.net:80 {
    log
    reverse_proxy /_matrix/* http://127.0.0.1:8008
    reverse_proxy /_synapse/client/* http://127.0.0.1:8008
    header /.well-known/* Access-Control-Allow-Headers "*"
}

matrix.cylindric.net:443 matrix.cylindric.net:8448 {
    log
    reverse_proxy http://127.0.0.1:8008

    tls mark@hanfordonline.co.uk
    tls {
      dns cloudflare APIKEY
    }
}
```


Final server config

```yaml
server_name: "cylindric.net"
pid_file: /data/homeserver.pid
public_baseurl: https://matrix.cylindric.net/
presence:
  presence_router:
allow_public_rooms_without_auth: false
allow_public_rooms_over_federation: false
listeners:
  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    resources:
      - names: [client, federation]
        compress: false
limit_remote_rooms:
retention:
acme:
    enabled: false
    port: 80
    bind_addresses: ['::', '0.0.0.0']
    reprovision_threshold: 30
    domain: matrix.example.com
    account_key_file: /data/acme_account.key
caches:
   per_cache_factors:
database:
  name: sqlite3
  args:
    database: /data/homeserver.db
log_config: "/data/matrix.cylindric.net.log.config"
media_store_path: "/data/media_store"
url_preview_accept_language:
recaptcha_public_key: "redacted"
recaptcha_private_key: "redacted"
enable_registration_captcha: true
enable_registration: true
registrations_require_3pid:
  - email
registration_shared_secret: "redacted"
allow_guest_access: false
account_threepid_delegates:
auto_join_rooms:
  - "#general:cylindric.net"
auto_join_rooms_for_guests: false
account_validity:
metrics_flags:
report_stats: true
room_prejoin_state:
macaroon_secret_key: "redacted"
form_secret: "redacted"
signing_key_path: "/data/matrix.cylindric.net.signing.key"
old_signing_keys:
trusted_key_servers:
  - server_name: "matrix.org"
saml2_config:
  sp_config:
  user_mapping_provider:
    config:
oidc_providers:
cas_config:
sso:
password_config:
   policy:
ui_auth:
email:
  smtp_host: redacted
  smtp_port: redacted
  smtp_user: "redacted"
  smtp_pass: "redacted"
  require_transport_security: redacted
  notif_from: "redacted"
password_providers:
push:
spam_checker:
user_directory:
stats:
opentracing:
redis:
experimental_features:
  spaces_enabled: true
```