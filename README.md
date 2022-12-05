# OksaZh_platform
OksaZh Platform repository

Name:             web
Namespace:        default
Address:          172.17.255.3
Default backend:  default-http-backend:80 (<none>)
Rules:
Host  Path  Backends
----  ----  --------
*
        /web   web-svc:8000 (172.17.0.7:8000,172.17.0.8:8000,172.17.0.9:8000)