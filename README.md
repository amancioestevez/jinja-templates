# jinja-templates

Plantillas Jinja2 para Home Assistant.

## Despliegue (Jenkins)

Pipeline configurado para desplegar a Home Assistant vía SSH/rsync usando la librería compartida `jenkins-shared-lib`.

Parámetros del job:

- `HA_HOST`: host/IP de Home Assistant (por defecto `192.168.1.150`).
- `HA_SSH_PORT`: puerto SSH del add-on (por defecto `2222`).
- `HA_CRED_ID`: ID de la credencial SSH en Jenkins (por defecto `ha_ssh_creds`).
- `DRY_RUN`: simular sin copiar (`rsync -n`).
- `MIRROR`: modo espejo, borra extras en destino (`--delete`).

Destino en HA:

- `DST_DIR=/config/custom_templates`

El Jenkinsfile usa el step `haCopy.rsync`:

```groovy
haCopy.rsync(
  host: env.HA_HOST,
  port: env.HA_SSH_PORT,
  userCredentialsId: params.HA_CRED_ID,
  srcDir: env.WORKSPACE,
  dstDir: env.DST_DIR,
  dryRun: params.DRY_RUN,
  mirror: params.MIRROR,
  include: ['*.jinja'],
  exclude: ['*']
)
```

Requisitos:

- `rsync` en el agente Jenkins y en el add-on de HA.
- Credencial SSH (usuario del add-on, típicamente `root`).
