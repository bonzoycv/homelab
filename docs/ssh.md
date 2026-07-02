## Ubuntu Server

Se instaló OpenSSH Server.

Se comprobó:

```bash
systemctl status ssh
```

---

## Acceso desde PikaOS

Alias:

```
ssh homelab
```

---

## Acceso desde Fedora

Se copiaron las claves SSH existentes.

También se reutilizó el archivo:

~/.ssh/config

Con ello ambos equipos administran exactamente la misma infraestructura.
