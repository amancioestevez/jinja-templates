# jinja-templates

Plantillas Jinja2 (macros reutilizables) para Home Assistant. Se despliegan en
`/config/custom_templates` y se importan desde cualquier plantilla de HA con
`{% from '<fichero>.jinja' import <macro> %}`.

> Tras desplegar, HA carga los cambios con *Herramientas de desarrollador → YAML → Recargar
> plantillas personalizadas* (servicio `homeassistant.reload_custom_templates`).

## Macros

### `common.jinja`

| Macro | Uso |
|---|---|
| `format_entity_value(entity_id, friendly_name)` | Una línea `- <nombre>: <estado> <unidad>` |
| `format_entity_values(items)` | Recorre `items()` (pasar `dict.items` sin llamar) y pinta una línea por entidad |

Ejemplo (automatización `telegram-bot-temp`):

```jinja
{% set area_temp = {
    'sensor.weather_salon_temperature': 'Salón',
    'sensor.weather_kitchen_temperature': 'Cocina',
} %}
{% from 'common.jinja' import format_entity_values %}
{{- format_entity_values(area_temp.items) }}
```

Consumidores: automatizaciones `telegram-bot-temp`, `telegram-bot-hum` y `telegram-bot-fuel`.

### `weather.jinja`

| Macro | Uso |
|---|---|
| `weather_condition_to_emoji(condition)` | Emoji para una condición de `weather.*` (`sunny`, `rainy`...); `❓` si no se conoce |
| `format_day_forecast(forecast_data, day_name)` | Bloque de un día (condición, máx., mín., prob. lluvia) en MarkdownV2 de Telegram |
| `forecast_three_days(weather_forecast)` | Hoy, mañana y pasado a partir de la respuesta de `weather.get_forecasts` para `weather.meteo` |

`forecast_three_days` espera el `response_variable` de `weather.get_forecasts`
(`{'weather.meteo': {'forecast': [...]}}`). Los puntos se escapan (`\.`) para MarkdownV2.

### `temperature.jinja`

Vacío (reservado).

## Despliegue (Jenkins)

Pipeline configurado para desplegar a Home Assistant vía SSH/rsync usando la librería compartida `jenkins-shared-lib`.

Parámetros del job:

- `HA_HOST`: host/IP de Home Assistant (por defecto `192.168.1.150`).
- `HA_SSH_PORT`: puerto SSH del add-on (por defecto `2222`).
- `HA_CRED_ID`: ID de la credencial SSH en Jenkins (por defecto `ha_jenkins`).
- `DRY_RUN`: simular sin copiar (`rsync -n`).
- `MIRROR`: modo espejo, borra extras en destino (`--delete`). Por defecto `false`.

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

Solo se copian los `*.jinja` de la raíz del repo.

Requisitos:

- `rsync` en el agente Jenkins y en el add-on de HA.
- Credencial SSH (usuario del add-on, típicamente `root`).
- Host key de HA en el `known_hosts` del agente (`StrictHostKeyChecking=yes`).

La gestión del resto del servidor (automatizaciones, scripts...) se documenta en `../ha-server`.
