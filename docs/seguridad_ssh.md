Se configuró autenticación exclusivamente mediante clave pública.

Se copiaron las claves con:

```bash
ssh-copy-id
```

Posteriormente se modificó:

```
/etc/ssh/sshd_config
```

Parámetros principales:

```
PubkeyAuthentication yes

PasswordAuthentication no

PermitRootLogin prohibit-password
```

Finalmente se verificó que el acceso por contraseña estaba deshabilitado.
