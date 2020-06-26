---
author: yamila
title: Crontab y mails, muchos problemas y una solución
slug: crontab-y-mail
date: 2020-06-26 09:00:00+00:00
comments: true
tags:
- linux
- crontab
---

En este post explico mi experiencia configurando una tarea de cron con unos envíos de correos con Amazon SES. No es exactamente un tutorial, aunque terminaré explicando qué aproximación tomé.

<!-- more -->

En mi empresa hemos hecho un pequeña pequeña migración en la infraestructura de la que me he encargado yo. Una de las tareas consistía en automatizar unos backups y enviar un correo si algo iba mal.

### Los problemas

Así que hice un script de bash que generaba los backups, los comprimía (`lz4`) y los enviaba a un bucket de S3 (`s3cmd`). Si algo salía mal, debía enviar un mail. Cron envía los correos a través del postfix de la máquina que configuré con Amazon SES. Y en Amazon SES había habilitado un usuario de *sysadmin* para que enviara los correos. Para enviar correos con SES programáticamente, hay que validar un correo o un dominio entero, pero tiene que ser una entidad previamente validada.

Para confirmar que todo iba bien, hice una pequeña prueba en crontab:

```
12 9 * * * ls noexiste.txt
```

El retorno de ese comando sería distinto de 0 y crontab enviaría un correo. ¡Casi! Por defecto, los correos se envían desde `root@{hostname}` así que SES no lo iba a aceptar. Estuve mirando los logs de `/var/log/syslog` y `/var/log/mail.log` y buceando por internet vi alguna respuesta como [esta](https://community.webfaction.com/questions/11800/how-do-i-set-the-sender-address-for-mail-sent-from-my-cron-jobs) donde mencionaban configuraciones de `MAILTO` y `MAILFROM`. ¡Buena pinta! Aunque es un hilo de 2012. Pero después encontré este [otro hilo](https://bugs.launchpad.net/ubuntu/+source/cron/+bug/1750051) mucho más reciente y la esperanza volvió a mí. Así que reconfiguré la prueba:

```
MAILTO:sysadmin@kaleidos.net
MAILFROM:sysadmin@kaleidos.net
12 9 * * * ls noexiste.txt
```

Esta configuración por lo visto no tiene ningún sentido, porque crontab no interpreta la variable `MAILFROM`. Falló estrepitosamente. Así que crontab pone su propio `from` y *yo no he visto cómo cambiarlo en ningún sitio*. De hecho, todo lo que me he encontrado al respecto es que no se puede cambiar y los *workarounds* para sortear este escollo. Algunas de las propuestas eran:

1. **cambiar el hostname** a `kaleidos.net` de forma que se creara un correo `root@kaleidos.net`; en esta aproximación, tenía varias alternativas:
- por un lado, crear `root@kaleidos.net` como usuario de nuestro correo, no parece elegante
- validar el dominio `{hostname}.kaleidos.net` y crear `root@{hostname}.kaleidos.net`; no me parecía muy elegante, aunque llegó un momento en que decidí probar y terminé en un callejón sin salida porque el dominio no se verificaba de ninguna manera

2. **usar sendmail**: usando sendmail se pueden configurar el `from` y el `to` (y todo lo demás de hecho) de un correo, así que prometía. La idea era mandar la salida `stderr` de cron como entrada para el body de `sendmail`. Después de pegarme un rato con la sintaxis de `2>&1` para el `pipe` terminé con una construcción totalmente abigarrada, un `pipeline` de `echos` encadenados difícilmente mantenibles y trazables. Si fallase algo (y lo raro sería que no fallase) iba a ser difícil entender qué estaba pasando. Además de que me ponía en un escenario donde el envío de los mensajes de error eran el punto débil del proceso que debía avisarme de los errores. Mal también.

Llegada a este punto había dedicado 1 día entero a no configurar nada. A bote pronto me parecía algo trivial (aunque para mí fuera nuevo) y en un día sólo había descartado opciones inválidas, pero no estaba más cerca de resolverlo. (Falso: descartar opciones inválidas es aprendizaje y además ayuda a avanzar en las tareas, pero después de una jornada entera de opciones inválidas estaba más bien frustradita). El script de backups funcionaba a las mil maravillas, eso sí. Estuve tentada de dejarlo así, si total, ¡el script de backups funcionaba a las mil maravillas!


### La solución

La solución vino en forma de *workaround* pero que me deja mucho control del proceso. Se trata de asegurarme de que a crontab siempre le llega un código de retorno 0 para que no envíe correos, sino que los envío por otro lado. Para ello, hice un *wrapper* en Python que:

- lanza el comando de backups con `subprocess`
- recoge el resultado
- si es correcto, no hace nada
- si es incorrecto, manda el correo usando [este snippet](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/examples-send-using-smtp.html) de la documentación oficial de AWS

```python
def send_mail(error):
    # código de envío de correo. El valor de error se interpola en el body del correo

if __name__ == "__main__":
    res = subprocess.run(sys.argv[1:], capture_output=True)
    if res.returncode != 0:
        error = res.stderr.decode()
        send_email(error)
```

En crontab, en lugar de lanzar el comando de backups, lanzo este `ses-wrapper.py` de la siguiente forma:

```
12 9 * * * /usr/bin/python3 /path/to/ses-wrapper.py /path/to/backups.sh
```

Con este mecanismo, no necesito postfix (no era un problema en realidad), pero además es un *wrapper* fácilmente reutilizable para otros procesos. Así que después de un día y medio (tampoco es para tanto en realidad), encontré varias cosas que no pude hacer funcionar directamente con crontab y también una solución apañada que podré usar más veces.

¡Espero que os resulte útil!
