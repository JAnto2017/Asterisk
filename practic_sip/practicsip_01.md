# CONFIGURACIÓN BÁSICA DE ASTERISK USANDO SIP

1. Dial Plan, reglas de marcado, configuración &rarr; `/etc/asterisk/extensions.conf`.
2. Configuración de PJSIP (mejora de SIP) &rarr; `/etc/asterisk/pjsip.conf`
3. Configuración protocolo SIP &rarr; `/etc/asterisk/sip.conf`.
4. Configuración de IP &rarr; `/etc/asterisk/asterisk.conf`.
5. Configuración de protocolo IAX &rarr; `/etc/asterisk/iax.conf`.
6. Configuración de funcionalidades y módulos &rarr; `/etc/asterisk/modules.conf`.

---

## Práctica SIP 01

Entrar en modo CLI. Para ello se debe realizar conexión con el servidor usando PuTTY o la consola de Asterisk. Pasar al modo de _superuser_ con el comando `sudo su` y dedes la ruta `/usr/src/asterisk-20.13.0#` escribimos el comando `asterisk -rvvvvv`.

- Ejercutar `asterisk -rvvvvv`.
- En modo CLI> clic en TAB (muestra los comandos)

## Práctica SIP 02

Por defecto viene cargado PJSIP. Para cargar un módulo SIP se utiliza el comando `modules load mod_sip.so`. Para desactivarlo se utiliza el comando `modules unload mod_sip.so`.

- Ejercutar: `modules load chan_sip.so` para cargar un módulo.
- Ejercutar: `modules unload mod_sip.so` para quitar un módulo.
- Ejecutar: `sip show peers` para ver los peers conectados.

## Práctica SIP 03

Acceder al archivo `/etc/asterisk/modules.conf` y realizar las siguientes modificaciones:

- En la línea de código `noload = chan_sip.so` comentar añadiendo un punto y coma al inicio de la misma.
- Añadimos la siguiente línea en su lugar: `noload = chan_pjsip.so`.
- Guardar archivo.

## Práctica SIP 04

Configuración de SIP en el archivo `/etc/asterisk/sip.conf`.

- Realizar una copia del fichero `sip.conf` a `sip.conf.old`. Para ello usar el comando `mv sip.conf sip.conf.old`.
- Abrir el fichero `sip.conf` con el editor de texto vim o nano.

Las configuraciones se encuentran divididas en parámetros (_valor_ <==> _parámetro_) divididas en secciones:

```sip.conf
[general]               ;seccion general

[2000]                  ;puede ser un nombre, es la sección de las extensiones
type = friend           ;identifica por IP y nombre
host = dynamic
secret = 1234
context = prueba
disallow = all
allow = ulaw
allow = alaw

;repetir para otras extensiones cambiando el número
```

- En modo CLI> `sip reload`.
- En modo CLI> `sip show peers`.

Muestra las extensiones creadas en el archivo `/etc/asterisk/sip.conf`.

## Práctica SIP 05

Las reglas de marcado están en el archivo: `/etc/asterisk/extensions.conf` y se pueden modificar con el editor de texto vim o nano.

Crear un archivo nuevo y dejar el original intacto.

- Realizar una copia del fichero `extensions.conf` a `extensions.conf.old`. Para ello usar el comando `mv extensions.conf extensions.conf.old`.
- Abrir el fichero `extensions.conf` con el editor de texto vim o nano.
- Guardar el archivo.

## Práctica SIP 06

Añadir las reglas de marcada en el archivo `/etc/asterisk/extensions.conf`:

```extensions.conf
[general]

;el plan de marcado se encuentra dividido en contextos
;exten => número_marcado, prioridad, aplicación

[prueba]
exten => 2000,1,Answer()
exten => 2000,2,Playback(demo-congrats) ;fichero de audio sin extensión
exten => 2000,3,Hangup()
```

Alternativa, cuando existen muchas líneas se puede usar n en lugar del número.

```extensions.conf
[general]

[prueba]
exten => 2000,1,Answer()
exten => 2000,n,Playback(demo-congrats) ;fichero de audio sin extensión
exten => 2000,n,Hangup()
```

Existe la posibilida de utilizar el comando `same` para crear un patrón de llamadas.

```extensions.conf
[general]

[prueba]
exten => 2000,1,Answer()
same => n,Playback(demo-congrats) ;fichero de audio sin extensión
same => n,Hangup()

```

- Recargar el dialplan con el comando `CLI> sip reload` y `CLI> dialplan reload`.
- Al marcar el número 2000 se reproduce el audio `demo-congrats` y se descuelga la llamada.
- Los ficheros de audio se encuentran en el directorio `/var/lib/asterisk/sounds/es/`. En esta carpeta están los audios en español con extensión _gsm_, _alaw_ y _ulaw_.

## Práctica SIP 07

Usando la aplicación `Dial(canal, timeout, opciones, URI)`. Sirve para que una extensión pueda llamar a otra extensión. La configuración se realiza en el archivo `/etc/asterisk/extensions.conf`:

```extensions.conf
[general]

[prueba]
exten => 2000,1,Dial(SIP/2000)
exten => 2001,1,Dial(SIP/2001)
exten => 2002,1,Dial(SIP/2002)
```

- Recargar el dialplan con el comando `CLI> sip reload` y `CLI> dialplan reload`.
- Al marcar el número 2000 se llama a la extensión 2000. Se pueden llamar entre todos los números de extensión definidos.
- Si se marca un número que no está definido, sale el error _Not found in context_.

## Práctica SIP 08

Desviar una llamada, tras un tiempo de por ejemplo 10 segundos.

```extensions.conf
[general]

[prueba]
exten => 2000,1,Dial(SIP/2000, 10, m)   ;m indica número que llama
same => n,Dial(SIP/2001, 10)
same => n,Playback(hello-world)         ;fichero de audio sin extensión
same => n,Hangup()
```

- Al llamar a la extensión _2000_ desde una extensión, p. ej., 2005, tras 10 segundos desvía la llamada a la extensión 2001 y reproduce un audio.

## Práctica SIP 09

Manejando la aplicación _Record()_ para la grabación de audios. La configuración se realiza en el archivo `/etc/asterisk/extensions.conf`:

```extensions.conf
[general]

[prueba]
exten => 808,1,Record(ejemplo.gsm,3)    ;3" indica el tiempo de grabación

exten => 809,1,Answer()
same => n,Playback(ejemplo)         ;fichero de audio sin extensión
same => n,Hangup()
```

- Marcamos el número 808 y se graba un audio de 3 segundos. Para finalizar la grabación se pulsa `#`.
- Marcamos el número 809 y se reproduce el audio grabado y se descuelga la llamada.
- `*CLI> core show applications record` para ver las grabaciones.

## Práctica SIP 10

Reproducir un audio y esperar hasta que se marque un dígito.

```extensions.conf
[general]

[prueba]
exten => 807,1,Record(invalido.gsm,3)   ;para grabar audio de error
exten => 808,1,Record(muchotiempo.gsm,3)   ;para grabar audio de timepo excedido

exten => 501,1,Goto(menu,inicio,1)

[menu]
exten => inicio,1,Background(ejemplo)
same => n,WaitExten(5)

exten => 1,1,SayDigits(1)
exten => 2,1,SayDigits(2)
exten => 3,1,SayDigits(3)
exten => 4,1,SayDigits(4)
exten => 5,1,SayDigits(5)
exten => 6,1,SayDigits(6)

exten => i,1,Playback(invalido)        ;fichero de audio sin extensión
exten => t,1,Playback(muchotiempo)         ;fichero de audio sin extensión
```

- Con la opción **i** tratamos reproduce un audio de error. Si se pulsa un número distinto a 1, 2, 3, 4, 5 o 6.
- Con la opcion **t** tratamos de reproducir un audio de error. Si se tarda más de
- `*CLI> core show applications record` para ver las grabaciones.
- `*CLI> dialplan show prueba` para ver el dialplan.

## Práctica SIP 11

Usar los prefijos en el Dialplan.

```extensions.conf
[general]

[prueba]
exten => 2000,1,Dial(SIP/${EXTEN},40,tT)
exten => _2XXX,1,Dial(SIP/${EXTEN},40,tT)
```

- X = 0 a 9
- Z = 1 a 9
- N = 2 a 9
- _2 = empieza con 2
- _2[345]X = empieza con 2 y tiene tres dígitos y comprendidos entre 0 y 9.

---

## Práctica SIP 12 - Combinación de varios módulos

Crear una configuración básica para Asterisk usando SIP en la que tres terminales SIP puedan llamarse entre sí.

### Archivos de configuración

- `sip.conf` &rarr; definición de los terminales SIP (usuarios)
- `extensions.conf` &rarr; definición de las reglas de marcado (dialplan) para que puedan llamarse en sí.

### Configuración de 3 terminales SIP - Archivo sip.conf

```sip.conf
[general]
context=internal
allowguest=no
allowoverlap=no
bindport=5060
bindaddr=0.0.0.0
srvlookup=yes

; === Terminal 101 ===
[101]
type=friend
host=dynamic
secret=clave101
context=internal
disallow=all
allow=ulaw
callerid="Terminal 101" <101>

; === Terminal 102 ===
[102]
type=friend
host=dynamic
secret=clave102
context=internal
disallow=all
allow=ulaw
callerid="Terminal 102" <102>

; === Terminal 103 ===
[103]
type=friend
host=dynamic
secret=clave103
context=internal
disallow=all
allow=ulaw
callerid="Terminal 103" <103>

```

### Llamadas entre extensión - Configuración del Archivo extensions.conf

```extensions.conf
[internal]
exten => 101,1,Dial(SIP/101)
 same => n,Hangup()

exten => 102,1,Dial(SIP/102)
 same => n,Hangup()

exten => 103,1,Dial(SIP/103)
 same => n,Hangup()

```

### Registrar los terminales

Se deben registrar los terminales _softphones como:

- Zoiper
- MicroSIP
- Linphone
- Teléfonos IP físicos.

En cada terminal se debe configurar de la siguiente forma:

| **Terminal** | **Usuario** | **Secret** | **Servidor** |
| --- | --- | --- | --- |
| 101 | 101 | clave101 | IP Asterisk |
| 102 | 102 | clave102 | IP Asterisk |
| 103 | 103 | clave103 | IP Asterisk |

### Verificación

- `sudo asterisk -rvvv`.
- En modo CLI> `sip show peers`.
- Realizar llamadas entre los terminales.

### Buzón de voz

#### Archivos a modificar

- `voicemail.conf` → define los buzones de voz.
- `extensions.conf` → lógica de desvío al buzón y redirección.

#### Configuración del Archivo voicemail.conf

```voicemail.conf
[general]
format=wav49|gsm|wav
serveremail=asterisk@example.com
attach=yes
emailsubject=Mensaje nuevo de ${CALLERID}
emailbody=Has recibido un nuevo mensaje de voz.
; Usa tu email si deseas recibir mensajes por correo
; El envío de email necesita configuración adicional de postfix/sendmail

[internal]
101 => 1234,Terminal 101,correo101@example.com
102 => 1234,Terminal 102,correo102@example.com
103 => 1234,Terminal 103,correo103@example.com
```

- `1234`: clave para acceder al buzon.
- `correo@example.com`: correo del buzon.

#### Configuración del Archivo extensions.conf

```extensions.conf
[internal]
; === Extensión 101 ===
exten => 101,1,Dial(SIP/101,15)
 same => n,VoiceMail(101@internal,u) ; u = unavailable
 same => n,Hangup()

; === Extensión 102 ===
exten => 102,1,Dial(SIP/102,15)
 same => n,VoiceMail(102@internal,u)
 same => n,Hangup()

; === Extensión 103 ===
exten => 103,1,Dial(SIP/103,15)
 same => n,VoiceMail(103@internal,u)
 same => n,Hangup()

; Acceder al buzón desde cualquier extensión marcando 8500
exten => 8500,1,VoiceMailMain()
 same => n,Hangup()
```

El número `15` en `Dial()` representa 15 seggundos para contestar antes de redirigir al buzón.

### Verificación - Probar Buzón y Desvío

- Llamar a una extensión y no contestar, debería redirigirse al buzón.
- Desde un terminal, marcar 8500 para acceder al buzón.
  - Ingresar el número de extensión (ej. 101), luego la clave (ej. 1234)

---
