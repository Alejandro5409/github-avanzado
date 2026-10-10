# Memoria: CI/CD con GitHub Actions

## Objetivo

Implementar un flujo de integración y despliegue continuo con GitHub Actions,
respetando la protección de las ramas `develop` y `main`.

## Estructura de ramas

- `main`: rama de producción.
- `develop`: rama de integración.
- Ramas `feature/*`: ramas de trabajo para nuevas funcionalidades y correcciones.

Los cambios se integran mediante Pull Requests. No se realizan pushes directos
a las ramas protegidas.

## Integración continua: CI

El workflow `.github/workflows/ci.yml` se ejecuta cuando:

- Se abre, actualiza o reabre un Pull Request hacia `develop`.
- Se realiza un push a `develop`.
- Se ejecuta manualmente mediante `workflow_dispatch`.

### Jobs

1. **Actualizar README**
   - Se ejecuta en `ubuntu-latest`.
   - Añade y comprueba una línea en el README dentro del job.
   - No realiza push directo a `develop`, respetando la protección de rama.

2. **Validar token de desarrollo**
   - Se ejecuta en un runner self-hosted Windows.
   - Depende del primer job mediante `needs: update-readme`.
   - Compara el secret `DEV_TOKEN` con la variable local `LOCAL_DEV_TOKEN`.

## Despliegue continuo: CD

El workflow `.github/workflows/cd.yml` se ejecuta cuando:

- Se realiza un push a `main`.
- Se ejecuta manualmente mediante `workflow_dispatch`.

### Jobs

1. **Actualizar versión en README**
   - Se ejecuta en `ubuntu-latest`.
   - Sustituye `AppVersion-0` por `AppVersion-1` en el README.
   - Comprueba que la nueva versión está presente.

2. **Validar token de producción**
   - Se ejecuta en un runner self-hosted Windows.
   - Depende del primer job mediante `needs: bump-version`.
   - Compara el secret `PROD_TOKEN` con la variable local `LOCAL_PROD_TOKEN`.

## Pruebas realizadas

- Ejecución de CI mediante Pull Request hacia `develop`.
- Ejecución de CI mediante push a `develop`.
- Validación correcta de `DEV_TOKEN` en el runner self-hosted.
- Sincronización de `develop` hacia `main` mediante Pull Request.
- Ejecución automática de CD tras el merge a `main`.
- Ejecución manual de CD mediante `workflow_dispatch`.
- Prueba de fallo controlado al modificar temporalmente `LOCAL_PROD_TOKEN`.
- Comprobación de que el workflow falla cuando los tokens no coinciden.
- Restauración del valor original y verificación de que CD vuelve a funcionar.

## Resultado

El sistema implementa un flujo completo de CI/CD con validación de secretos,
ejecución en distintos sistemas operativos, dependencias entre jobs y respeto
de las reglas de protección de ramas.
