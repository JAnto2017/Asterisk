# PRÁCTICAS UTILIZANDO PJSISP

## Práctica PJSIP 01

### Archivos clave para chan_pjsip

1. `pjsip.conf` &rarr; donde se definen los _endpoints_, _aors_ y _auths_.
2. `extensions.conf` &rarr; el _dialplan_ para permitir llamadas entre las extensiones.

### Configurar 4 Extensiones básicas

Las extesniones creadas en el fichero `pjsip.conf` tienen los números: 101 a 104.

```asterisk
[global]
;type=global
;user_agent=Asterisk-PJSIP

[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060
;bind=0.0.0.0

; === EXTENSION 101 ================================
[101]
type=endpoint
transport=transport-udp
context=llamadas-internas
disallow=all
allow=alaw
allow=ulaw
auth=auth101
aors=101

[auth101]
type=auth
auth_type=userpass
username=101
password=123456789

[101]
type=aor
max_contacts=1
remove_existing=yes

; === EXTENSION 102 ================================
[102]
type=endpoint
transport=transport-udp
context=llamadas-internas
disallow=all
allow=alaw
allow=ulaw
auth=auth102
aors=102

[auth102]
type=auth
auth_type=userpass
username=102
password=clave102

[102]
type=aor
max_contacts=1

; === EXTENSION 103 ================================
[103]
type=endpoint
transport=transport-udp
context=llamadas-internas
disallow=all
allow=alaw
allow=ulaw
auth=auth103
aors=103

[auth103]
type=auth
auth_type=userpass
username=103
password=clave103

[103]
type=aor
max_contacts=1

; === EXTENSION 104 ================================
[104]
type=endpoint
transport=transport-udp
context=llamadas-internas
disallow=all
allow=alaw
allow=ulaw
auth=auth104
aors=104

[auth104]
type=auth
auth_type=userpass
username=104
password=clave104

[104]
type=aor
max_contacts=1

```

### Permitir llamadas entre las extensiones

La configuración del dialplan se encuentra en el fichero `/et/asterisk/extensions.conf`. Este fichero debe tener la siguiente configuración para poder realizar llamadas entre las extensiones:

```asterisk
[general]
static=yes

[globals]

[llamadas-internas]
exten => 101,1,Dial(PJSIP/${EXTEN})
 same => n,Hangup()

exten => 102,1,Dial(PJSIP/${EXTEN})
 same => n,Hangup()

exten => 103,1,Dial(PJSIP/${EXTEN})
 same => n,Hangup()

exten => 104,1,Dial(PJSIP/${EXTEN})
 same => n,Hangup()
```

### Registrar terminales

Puedes usar softphones como _Zoiper_, _MicroSIP_, _Linphone_ o _Teléfonos IP físicos_. La configuración de cada terminal es la siguiente:

| **Terminal** | **Usuario** | **Secret** | **Servidor** | **Puerto** |
| --- | --- | --- | --- | --- |
| 101 | 101 | clave101 | IP Asterisk | 5060 |
| 102 | 102 | clave102 | IP Asterisk | 5060 |
| 103 | 103 | clave103 | IP Asterisk | 5060 |
| 104 | 104 | clave104 | IP Asterisk | 5060 |

### Verificación

- `sudo asterisk -rvvv`.
- En modo CLI> `core reload`.
- En modo CLI> `pjsip reload`.
- En modo CLI> `pjsip show ...` añadir modificador. Para ver los disponibles, pulsar la tecla TAB.
- Realizar llamadas entre los terminales.

- - -

## Buzón de Voz y Desvío de llamadas al buzón de voz

### Archivos a modificar

1. `pjsip.conf` &rarr; el _dialplan_ para permitir llamadas entre las extensiones.
2. `extensions.conf` &rarr; el _dialplan_ para permitir llamadas al buzón de voz.
3. `voicemail.conf` → define los buzones de voz.

### Configuración buzones de voz

Editar el archivo `voicemail.conf`:

```asterisk
[general]
format=wav49|gsm|wav
serveremail=asterisk@example.com
attach=yes
emailsubject=Mensaje nuevo de ${CALLERID}
emailbody=Tienes un nuevo mensaje de voz.

[internal]
101 => 1234,Terminal 101,correo101@example.com
102 => 1234,Terminal 102,correo102@example.com
103 => 1234,Terminal 103,correo103@example.com
104 => 1234,Terminal 104,correo104@example.com
```

La contraseña `1234` es para acceder al buzón. Puedes cambiar los correos si deseas enviar mensajes por email (requiere configuración adicional de correo en Asterisk).

### Desvío y acceso al buzón de voz

Actualizando el contexto `[internal]` en el archivo `extensions.conf`, para que desvíe al buzón si no contestan en 15 segundos:

```asterisk
[internal]
exten => 101,1,Dial(PJSIP/101,15)
 same => n,VoiceMail(101@internal,u)
 same => n,Hangup()

exten => 102,1,Dial(PJSIP/102,15)
 same => n,VoiceMail(102@internal,u)
 same => n,Hangup()

exten => 103,1,Dial(PJSIP/103,15)
 same => n,VoiceMail(103@internal,u)
 same => n,Hangup()

exten => 104,1,Dial(PJSIP/104,15)
 same => n,VoiceMail(104@internal,u)
 same => n,Hangup()

; Acceso al buzón de voz
exten => 8500,1,VoiceMailMain()
 same => n,Hangup()
```

Cambia 15 si quieres más o menos segundos antes del desvío.

En `VoiceMailMail(...,u)` reproduce el mensaje de "usuario no disponible".

### Verificación - Probar Buzón y Desvío

- Llamar a una extensión y no contestar, debería redirigirse al buzón.
- Desde un terminal, marcar 8500 para acceder al buzón.
- Ingresar el número de extensión (ej. 101), luego la clave (ej. 1234)
- Realizar llamadas entre las extensiones.
