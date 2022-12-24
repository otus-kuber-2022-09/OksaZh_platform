Домашнее задание 11

1. Создан кластер kind

cd kubernetes-vault
kind create cluster --config cluster.yml

2. Инсталляция hashicorp vault HA в k8s

Склонирован репозиторий consul 

git clone https://github.com/hashicorp/consul-helm.git
helm install consul consul-helm

Склонирован репозиторий vault

git clone https://github.com/hashicorp/vault-helm.git 

Заданы переменные в kubernetes-vault/vault/values.yml:

helm install vault vault-helm -f kubernetes-vault/vault/values.yml

helm status vault 

NAME: vault
LAST DEPLOYED: Mon Dec 19 22:25:01 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

$ helm status vault
$ helm get vault

3. Инициализация vault:

kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1

Unseal Key 1: ZcjqlYKKvbnZk9JVYD+iRw59uOhZJqM6KFBDI1ZRJbA=

Initial Root Token: s.Oldlfv9RSLeTesH9ciypFQHZ

Состояние vault'а:

kubectl exec -it vault-0 -- vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       1
Threshold          1
Unseal Progress    0/1
Unseal Nonce       n/a
Version            1.3.1
HA Enabled         true

4. Распечатаем vault

kubectl exec -it vault-0 -- vault operator unseal 'ZcjqlYKKvbnZk9JVYD+iRw59uOhZJqM6KFBDI1ZRJbA='
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.3.1
Cluster Name           vault-cluster-b235a4c2
Cluster ID             721389d8-7aba-8644-d8d0-45f5da9f7c9a
HA Enabled             true
HA Cluster             n/a
HA Mode                standby
Active Node Address    <none>

kubectl exec -it vault-1 -- vault operator unseal 'ZcjqlYKKvbnZk9JVYD+iRw59uOhZJqM6KFBDI1ZRJbA='
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.3.1
Cluster Name           vault-cluster-b235a4c2
Cluster ID             721389d8-7aba-8644-d8d0-45f5da9f7c9a
HA Enabled             true
HA Cluster             https://10.244.3.4:8201
HA Mode                standby
Active Node Address    http://10.244.3.4:8200

kubectl exec -it vault-2 -- vault operator unseal 'ZcjqlYKKvbnZk9JVYD+iRw59uOhZJqM6KFBDI1ZRJbA='
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.3.1
Cluster Name           vault-cluster-b235a4c2
Cluster ID             721389d8-7aba-8644-d8d0-45f5da9f7c9a
HA Enabled             true
HA Cluster             https://10.244.3.4:8201
HA Mode                standby
Active Node Address    http://10.244.3.4:8200

Просмотр списка доступных авторизаций

kubectl exec -it vault-0 -- vault auth list

Error listing enabled authentications: Error making API request.

Т.к. нет токена для доступа к API под пользователем vault т.к. мы не залогинены.

5. Логин в vault

kubectl exec -it vault-0 -- vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.Oldlfv9RSLeTesH9ciypFQHZ
token_accessor       p1Goswabcmdlvk9w9aDLRZ1M
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]

Повторный запрос списка авторизаций: 

kubectl exec -it vault-0 -- vault auth list
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_8fb86105    token based credentials


6. Создание секретов

kubectl exec -it vault-0 -- vault secrets enable --path=otus kv
kubectl exec -it vault-0 -- vault secrets list --detailed
kubectl exec -it vault-0 -- vault kv put otus/otus-ro/config username='otus'  password='asajkjkahs'
kubectl exec -it vault-0 -- vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'
kubectl exec -it vault-0 -- vault read otus/otus-ro/config
kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config


Чтение секретов:

kubectl exec -it vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus

kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus

7. Включена авторизация черерз k8s

kubectl exec -it vault-0 -- vault auth enable kubernetes

kubectl exec -it vault-0 -- vault auth list
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_34c79702    n/a
token/         token         auth_token_8fb86105         token based credentials

Создан yaml для ClusterRoleBinding:

Применение Service Account vault-auth и ClusterRoleBinding:

kubectl create serviceaccount vault-auth
serviceaccount/vault-auth created

kubectl apply --filename vault-auth-service-account.yml
clusterrolebinding.rbac.authorization.k8s.io/role-tokenreview-binding created

Переменные для записи в конфиг кубер авторизации

export VAULT_SA_NAME=$(kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}")
export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" |
base64 --decode; echo)
export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" |
base64 --decode; echo)
export K8S_HOST=$(more ~/.kube/config | grep server |awk '/http/ {print $NF}')
### alternative way
export K8S_HOST=$(kubectl cluster-info | grep ‘Kubernetes master’ | awk ‘/https/ {print
$NF}’ | sed ’s/\x1b\[[0-9;]*m//g’ )

sed 's/\x1b\[[0-9;]*m//g' убирает цветовые коды

Запись конфиг в vault

kubectl exec -it vault-0 -- vault write auth/kubernetes/config \
token_reviewer_jwt="$SA_JWT_TOKEN" \
kubernetes_host="$K8S_HOST" \
kubernetes_ca_cert="$SA_CA_CRT"

Success! Data written to: auth/kubernetes/config

Файл политики:

tee otus-policy.hcl <<EOF
path "otus/otus-ro/*" {
capabilities = ["read", "list"]
}
path "otus/otus-rw/*" {
capabilities = ["read", "create", "list"]
}
EOF

Политика и роль в vault

kubectl cp otus-policy.hcl vault-0:/tmp
kubectl exec -it vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl
Success! Uploaded policy: otus-policy

kubectl exec -it vault-0 -- vault write auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=default policies=otus-policy ttl=24h
Success! Data written to: auth/kubernetes/role/otus

Проверка авторизации

Создание пода с привязанным сервис аккаунтом и установка curl и jq

kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth --image alpine:3.7
Flag --generator has been deprecated, has no effect and will be removed in the future.
If you don't see a command prompt, try pressing enter.

apk add curl jq

Залогинимся и получим клиентский токен

VAULT_ADDR=http://vault:8200
KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}'
$VAULT_ADDR/v1/auth/kubernetes/login | jq
TOKEN=$(curl -k -s --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "test"}'
$VAULT_ADDR/v1/auth/kubernetes/login | jq '.auth.client_token' | awk -F\" '{p
rint $2}')

{
  "request_id": "7831e2bd-e678-9119-b3f7-414f11df0e74",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "s.oEkWr5EavQnmDbNWzbNjb3DV",
    "accessor": "tBubLc2z5y4gDTBPK4088cLa",
    "policies": [
      "default",
      "otus-policy"
    ],
    "token_policies": [
      "default",
      "otus-policy"
    ],
    "metadata": {
      "role": "otus",
      "service_account_name": "vault-auth",
      "service_account_namespace": "default",
      "service_account_secret_name": "vault-auth-token-hp7hk",
      "service_account_uid": "88951275-8de4-4071-a405-6358ea392ca5"
    },
    "lease_duration": 86400,
    "renewable": true,
    "entity_id": "61525b73-a617-6a6d-aabe-93159f89d93c",
    "token_type": "service",
    "orphan": true
  }
}

Прочитаем записанные ранее секреты и попробуем их обновить

    используем свой клиентский токен
    проверим чтение

#curl --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-ro/config
{"request_id":"8df66e11-e528-2e43-2dfe-30ee088db9c1","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}
# curl --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config
{"request_id":"98e739cd-406c-92fc-5f37-3828cc510222","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}

проверим запись

#curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-ro/config
{"errors":["1 error occurred:\n\t* permission denied\n\n"]}
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config
{"errors":["1 error occurred:\n\t* permission denied\n\n"]}
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config1
write ok

Причина ошибки связана с политикой

path "otus/otus-ro/*" {
  capabilities = ["read", "list"]
}
path "otus/otus-rw/*" {
  capabilities = ["read", "create", "list"]
}

В otus/otus-rw есть create, но update нет.

Обновление политики otus-policy.hcl и применение изменений

kubectl cp otus-policy.hcl vault-0:/tmp
kubectl exec -it vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl
Success! Uploaded policy: otus-policy

Проверка записи

curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config
{"request_id":"56e7b20e-8867-efb0-5c1d-a190aa16d9d3","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"bar":"baz"},"wrap_info":null,"warnings":null,"auth":null}


8. Use case использования авторизации через кубер

Репозиторий с примерами

git clone https://github.com/hashicorp/vault-guides.git
cd vault-guides/identity/vault-agent-k8s-demo


В каталоге configs-k8s скорректированы конфиги с учетом ранее созданых ролей и секретов
Проверен и скорректирован конфиг example-k8s-spec.yml

Запуск примера

kubectl apply -f configmap.yaml

configmap/example-vault-agent-config created

kubectl get configmap example-vault-agent-config -o yaml
apiVersion: v1
data:
  consul-template-config.hcl: |
    vault {
      renew_token = false
      vault_agent_token_file = "/home/vault/.vault-token"
      retry {
        backoff = "1s"
      }
    }

    template {
    destination = "/etc/secrets/index.html"
    contents = <<EOT
    <html>
    <body>
    <p>Some secrets:</p>
    {{- with secret "otus/otus-ro/config" }}
    <ul>
    <li><pre>username: {{ .Data.username }}</pre></li>
    <li><pre>password: {{ .Data.password }}</pre></li>
    </ul>
    {{ end }}
    </body>
    </html>
    EOT
    }
  vault-agent-config.hcl: |
    exit_after_auth = true

    pid_file = "/home/vault/pidfile"

    auto_auth {
        method "kubernetes" {
            mount_path = "auth/kubernetes"
            config = {
                role = "otus"
            }
        }

        sink "file" {
            config = {
                path = "/home/vault/.vault-token"
            }
        }
    }
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"consul-template-config.hcl":"vault {\n  renew_token = false\n  vault_agent_token_file = \"/home/vault/.vault-token\"\n  retry {\n    backoff = \"1s\"\n  }\n}\n\ntemplate {\ndestination = \"/etc/secrets/index.html\"\ncontents = \u003c\u003cEOT\n\u003chtml\u003e\n\u003cbody\u003e\n\u003cp\u003eSome secrets:\u003c/p\u003e\n{{- with secret \"otus/otus-ro/config\" }}\n\u003cul\u003e\n\u003cli\u003e\u003cpre\u003eusername: {{ .Data.username }}\u003c/pre\u003e\u003c/li\u003e\n\u003cli\u003e\u003cpre\u003epassword: {{ .Data.password }}\u003c/pre\u003e\u003c/li\u003e\n\u003c/ul\u003e\n{{ end }}\n\u003c/body\u003e\n\u003c/html\u003e\nEOT\n}\n","vault-agent-config.hcl":"exit_after_auth = true\n\npid_file = \"/home/vault/pidfile\"\n\nauto_auth {\n    method \"kubernetes\" {\n        mount_path = \"auth/kubernetes\"\n        config = {\n            role = \"otus\"\n        }\n    }\n\n    sink \"file\" {\n        config = {\n            path = \"/home/vault/.vault-token\"\n        }\n    }\n}\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"example-vault-agent-config","namespace":"default"}}
  creationTimestamp: "2022-21-12T16:20:41Z"
  name: example-vault-agent-config
  namespace: default
  resourceVersion: "603374"
  selfLink: /api/v1/namespaces/default/configmaps/example-vault-agent-config
  uid: 905b0e4b-b142-4d3b-a00c-7f1ef3eca0ba

kubectl apply -f example-k8s-spec.yaml

pod/vault-agent-example created

Проверка

Коннект к поду nginx. index.html:

kubectl exec -ti vault-agent-example -c nginx-container  -- cat /usr/share/nginx/html/index.html
<html>
<body>
<p>Some secrets:</p>
<ul>
<li><pre>username: otus</pre></li>
<li><pre>password: asajkjkahs</pre></li>
</ul>
</body>
</html>

Создадим CA на базе vault:

kubectl exec -it vault-0 -- vault secrets enable pki
Success! Enabled the pki secrets engine at: pki/

kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki
Success! Tuned the secrets engine at: pki/

kubectl exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal \
common_name="example.com" \
ttl=87600h > CA_cert.crt

пропишем урлы для ca и отозванных сертификатовсертификатов

kubectl exec -it vault-0 -- vault write pki/config/urls issuing_certificates="http://vault:8200/v1/pki/ca" сrl_distribution_points="http://vault:8200/v1/pki/crl"

Success! Data written to: pki/config/urls

Создадим промежуточный сертификат:

kubectl exec -it vault-0 -- vault secrets enable -path=pki_int pki
Success! Enabled the pki secrets engine at: pki_int/

kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=43800h pki_int
Success! Tuned the secrets engine at: pki_int/

kubectl exec -it vault-0 -- vault write -format=json pki_int/intermediate/generate/internal \
common_name="example.com Intermediate Authority" \
| jq -r '.data.csr' > pki_intermediate.csr

пропишем промежуточный сертификат в vaultvault

kubectl cp pki_intermediate.csr vault-0:./
kubectl exec -it vault-0 -- vault write -format=json pki/root/sign-intermediate \
csr=@pki_intermediate.csr \
format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem
kubectl cp intermediate.cert.pem vault-0:./
kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed
certificate=@intermediate.cert.pem

Создадим и отзовем новые сертификаты

Создадим роль для выдачи сертификатов:

kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru allowed_domains="example.ru" allow_subdomains=true max_ttl="720h"
Success! Data written to: pki_int/roles/example-dot-ru

Создадим и отзовем сертификат:

kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUDuxUYhUe7OEOyz8RUzT0xQtBI+kwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMTA5MjQxODIxNDlaFw0yNjA5
MjMxODIyMTlaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANV44AD4AXJx
z8u7b7Y+LJiyyEPPm+m5EgtemrOiQDFBV6/cNK4WntUwFRxM/8CrpP/aViVoHlUb
by9DtbacjLI7dRvVkuGZVm3+SmsdSge9tn2koTlhHDu62SbaRRK3l1u0YsaCZt7D
56F/m//IsNJQSMSpQG+pS2mUJOFqCQosEgcPRxYWuy9S8i7wHZtVyTDNRqIMzx8/
sEGA5Ax23P96O0siUN4azbf3+UI2B37LfvIPfO4ufr+f63oEfvUX6clrIMwqZTzA
Oss1iu3IwVsZDymUVrlKEp1pwp22Gppxd8+1m68uZXjAnDcS7CeI5/2w3Jd56vEZ
XSluDTBCeU8CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQU0Us447XWKp/2jjDOn6ZLQKnd79swHwYDVR0jBBgwFoAU
XzzfSYUeH+tkDdaWZwYzs6Di3JcwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
nEX3iE8LhM5rxtRoZch7iFPv9tXTj8VAB7ItVtvWX1mS2DreGUysPYBSdK4QvPOq
v4BF3wLAnTC+0M50pw0fAMI9icjVNpNTCQBihd0QXuvW/5wKBCMVDpMknIjjNt7S
I2nCnJ9+xTGHrTvUCW7F5z0XBOZT3zRNk1VUvjpPskfKHb2Z3ZFcIpzUOMJrRXKL
SYIhNq5w9inWy1YIv0Zi10+2+n3Fuusz+7p71vT5BrtR1nlUUvtNpmOjokYzk4d3
Oe6LuJBEs0E4rWlOH0WXDswdTmMNISfqyyrixk58aiUxeki/3aPEtu1JFtyYYb9P
+LNeqWapMWq8kb0pQLaz/A==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUdbJDO03HgpbbihwWKPlLR4y5Qu0wDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIxMDkyNDE4MjgxOFoXDTIxMDkyNTE4Mjg0OFowHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDQ
EqXbfAia6LT7FQ/8tzVtsZXAFN+Jd2b/0StnsgfibibVZG1A6bY6Nit9Oep3Pxa7
h1Du7ijoYnYeW7MjqVrRPvGVkjG92wM5O3QMy+Dafr/3L0jOA2rxpqZM9aFK1bn0
dAclVXPePSOuRT0xyZW3nB15jXBLO2w/xPikont26ZTTwS+jdPDkNJKxyLSMBMXx
SpolnEPoiIHpaDyQtCv1ukZLd+2hcYLr9yVkBxbV6IA5HbbQ/QDD1GcA9Shunk/K
1ElGOtSD6Gqt6TzBRwgRKwh9rw6aTXWUJaKahQbYT5lQXShj/AWMuaj6qxpXdDQF
Mj8JRTdTfve/H2mHUEedAgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUVL3E+9kYx7FLX8U8
U3HyqrGcigkwHwYDVR0jBBgwFoAU0Us447XWKp/2jjDOn6ZLQKnd79swHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAIQ6DuBi
7u9zf1Ldl+ffVafAFnSHgkMzDy5YIlJBu/kgq5xHVR7sTOoj/6oveEdIjG6YxWnZ
g9C/8qAZL+8XAIkA2XPk0eUDYkOTzvp9H0ahh4Qw+vJPlfvkV8BW/vCBFPCcN6DM
3oruYfos8nTGoODQMIMTy6EUWBKpE8/sxvkH1lKqw5k14bGIJ9WLmafiOr5Njl4W
A+fQ1t/Jp8czwfznECaWL+RO7YzCAm5SPcrkmlh1Wyxj8qAq/5GHiLggAkgz9Yxd
XcYtqGKhB/S7ll0wofDysZ9Zds9VICSENMKaz6o+IpKC+jDfLGi/U6Ja0bhWoEya
6juzS8b93vBDV9A=
-----END CERTIFICATE-----
expiration          1632594528
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUDuxUYhUe7OEOyz8RUzT0xQtBI+kwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMTA5MjQxODIxNDlaFw0yNjA5
MjMxODIyMTlaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANV44AD4AXJx
z8u7b7Y+LJiyyEPPm+m5EgtemrOiQDFBV6/cNK4WntUwFRxM/8CrpP/aViVoHlUb
by9DtbacjLI7dRvVkuGZVm3+SmsdSge9tn2koTlhHDu62SbaRRK3l1u0YsaCZt7D
56F/m//IsNJQSMSpQG+pS2mUJOFqCQosEgcPRxYWuy9S8i7wHZtVyTDNRqIMzx8/
sEGA5Ax23P96O0siUN4azbf3+UI2B37LfvIPfO4ufr+f63oEfvUX6clrIMwqZTzA
Oss1iu3IwVsZDymUVrlKEp1pwp22Gppxd8+1m68uZXjAnDcS7CeI5/2w3Jd56vEZ
XSluDTBCeU8CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQU0Us447XWKp/2jjDOn6ZLQKnd79swHwYDVR0jBBgwFoAU
XzzfSYUeH+tkDdaWZwYzs6Di3JcwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
nEX3iE8LhM5rxtRoZch7iFPv9tXTj8VAB7ItVtvWX1mS2DreGUysPYBSdK4QvPOq
v4BF3wLAnTC+0M50pw0fAMI9icjVNpNTCQBihd0QXuvW/5wKBCMVDpMknIjjNt7S
I2nCnJ9+xTGHrTvUCW7F5z0XBOZT3zRNk1VUvjpPskfKHb2Z3ZFcIpzUOMJrRXKL
SYIhNq5w9inWy1YIv0Zi10+2+n3Fuusz+7p71vT5BrtR1nlUUvtNpmOjokYzk4d3
Oe6LuJBEs0E4rWlOH0WXDswdTmMNISfqyyrixk58aiUxeki/3aPEtu1JFtyYYb9P
+LNeqWapMWq8kb0pQLaz/A==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA0BKl23wImui0+xUP/Lc1bbGVwBTfiXdm/9ErZ7IH4m4m1WRt
QOm2OjYrfTnqdz8Wu4dQ7u4o6GJ2HluzI6la0T7xlZIxvdsDOTt0DMvg2n6/9y9I
zgNq8aamTPWhStW59HQHJVVz3j0jrkU9McmVt5wdeY1wSztsP8T4pKJ7dumU08Ev
o3Tw5DSSsci0jATF8UqaJZxD6IiB6Wg8kLQr9bpGS3ftoXGC6/clZAcW1eiAOR22
0P0Aw9RnAPUobp5PytRJRjrUg+hqrek8wUcIESsIfa8Omk11lCWimoUG2E+ZUF0o
Y/wFjLmo+qsaV3Q0BTI/CUU3U373vx9ph1BHnQIDAQABAoIBAQCdk4HIFsbtig6F
mA3jdVwhFrwyG5yunp6CXgZhIZKXCJSgRs32uwgmTZ/h1lqatEyi+Hdyeyq/0tFh
bFDeUQNWNDUA8RZ6kcJ/NWdNyZkf353BtS2N10jGeU64Oc1Mv090seo3e9+kDulW
sVkGu4OG6dPomhTQ5M+1+5XSGLsn8Z/ZKqrKPS2TVJcILVX3NgtVP4d2Qf4yhenF
n0VZv2fJ7rtaJxqjU5co/HHnTfCIqXd7hRw9o/VblmPv84x75OfBr1LBInwYUrqS
+PKT/4wxc5oGFFN68d2L4rVvQMm26IvTCdjgboLBbRrVOBivb+SXOokSYTsVPc7q
ir6GEEQBAoGBAN7FQgoGs9PgIcIOxvRAq82Gbuo7GJMuRZb5Lf8LOcL45MMgEDCL
OHze+T7rbrpgd8WX2dNkrtJ8Aem5prmA7ZneNp6g/agDm6TmdnVUndNeDZy/1Iao
461+pJgGpJsveA3qM3cEHiH2yqZvbdw2JXkGnboVbGPJVxix92WrprpFAoGBAO8c
JFir5fZ5FhAI0rhZJnNTMwSkUT49IO+AG9BKh3EG84ZswRdRZ262kSfQPXVD9HEn
oyrgqwI2mI693BGXxaddjzOOWhKdRehpRDR1ev1RkY9tEVzI8nxV38alwMQr92LR
gHNCx2zUPhcp7J/Nvt1KA8PFlBgWKKzU58MR45l5AoGAD2UQYEMAUGcPzipZQ23o
sYZVyegVla4/7uP/cr2i2z96B6YCmGg2miKKlPeOKmEaRdRtoDc4AaHCPBWxWOZ5
BQYfPi0f+mltayLmEsurMH0ycZ+sHzYyrb2vwDXNUFAiesuxjMsDDhPRA1l1/R7c
zhVP9xkd6XNzimhaEXOgTQUCgYEA4NCx79k376TrtInHLmNL/rSkTGH+rSkmdWkb
PZ1FeWUSxTot1rHIMVVgZ3Goxz/sbhPZm2//+aXBjLxAVR5BTdpu0Qev8r6Cw0Fu
SnCHAfSWiqb+4yFgtLy9GPYxp4C7KeNXBYgtH0rzUi4t+BantUJpBcIYOwlilxXb
DxMbzukCgYEAkYQOeh0wHfZO/dRG2OHB6RwkEMg5ZiK8N3KpNB+VMVLBnWogpW6c
iWl8uRDDE7ef3uWHdGRkA7mrWAPYkhBoCp80WFkfOJSReoovJGjF+Bd3vO/hRwFC
pnPqo55LRHKKCmYqJbgi4pHRmvkiAS0Cu+BQ/wNGtjA7G5mIRJY4fIo=
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       75:b2:43:3b:4d:c7:82:96:db:8a:1c:16:28:f9:4b:47:8c:b9:42:ed

kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="75:b2:43:3b:4d:c7:82:96:db:8a:1c:16:28:f9:4b:47:8c:b9:42:ed"
Key                        Value
---                        -----
revocation_time            1632508156
revocation_time_rfc3339    2022-12-21T13:29:16.892443241Z