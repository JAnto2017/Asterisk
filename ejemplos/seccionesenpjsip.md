# Ejemplo de secciones en pjsip.conf

---

```asterisk
[101]
type=endpoint                   _;sección tipo endpoint_
context=extensiones-empresa     _;context del endpoint
disallow=all                    _;códec de voz
allow=alaw
aors=101
auth=auth101                    _;enlace a otras secciones
transport=capa-transports

[101]
type=aor                        _;sección tipo aor
max_contacts=1                  _;opcional
remove_existing=yes             _;número de endpoint permitidos

[auth101]
type=auth                       _;sección tipo auth
username=101                    
password=123456789
```

---
