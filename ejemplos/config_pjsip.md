# Ejemplo de configuración de pjsip.conf

---

```asterisk
;XXXX Definición del tipo de transporte XXXX

[capa-transporte]
type=transport
protocol=udp
bind=0.0.0.0

;XXXX Definición de las extensiones XXXX

[101]
type=endpoint
context=llamadas-internas
disallow=all
allow=alaw
aors=101
auth=auth101
transport=capa-transporte

[101]
type=aor
max_contacts=1
remove_existing=yes

[auth101]
type=auth
username=101
password=123456789101

;--------------------------------

[102]
type=endpoint
context=llamadas-internas
disallow=all
allow=alaw
aors=102
auth=auth102
transport=capa-transporte

[102]
type=aor
max_contacts=2
remove_existing=yes

[auth102]
type=auth
username=102
password=123456789102

;--------------------------------

[103]
type=endpoint
context=llamadas-internas
disallow=all
allow=alaw
aors=103
auth=auth103
transport=capa-transporte

[103]
type=aor
max_contacts=1
remove_existing=yes

[auth103]
type=auth
username=103
password=123456789103
```

---
