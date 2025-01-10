# Plantilla de Despliegue de n8n en Azure

Esta plantilla ARM permite desplegar n8n en Azure Container Instances (ACI) con SSL/TLS automático mediante Caddy y persistencia de datos a través de Azure File Share.

## Requisitos previos

- **Cuenta de Azure** con una suscripción activa.
- **Azure Container Registry** Crear un ACR y subir las siguientes imagenes de docker con los siguientes comandos: 
```bash
docker tag caddy:latest <nombre-del-registro>.azurecr.io/caddy:latest
docker push <nombre-del-registro>.azurecr.io/caddy:latest
docker tag n8nio/n8n:latest <nombre-del-registro>.azurecr.io/n8nio/n8n:latest
docker push <nombre-del-registro>.azurecr.io/n8nio/n8n:latest
```
## Componentes del Despliegue

- **n8n**: Contenedor principal que ejecuta la aplicación n8n.
- **Caddy**: Reverse proxy que proporciona:
  - Gestión automática de certificados SSL/TLS.
  - Enrutamiento HTTPS seguro.
  - Redirección automática de HTTP a HTTPS.
- **Azure File Share**: Almacenamiento persistente para los datos de n8n.

## Despliegue Rápido

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdavidcmeier%2Fn8n-deploy-azure%2Fmain%2Fdeploy-n8n.json)

## Parámetros de Configuración

La plantilla requiere los siguientes parámetros:

- `name`: Nombre del despliegue (5-63 caracteres).
  - Debe comenzar con una letra.
  - Solo puede contener letras minúsculas, números y guiones.
  - Se utilizará para la URL: `<name>.<region>.azurecontainer.io`.
- `storagePrefix`: Prefijo para el nombre de la cuenta de almacenamiento (máx. 10 caracteres, por defecto: "n8n").
- `n8nUsername`: Nombre de usuario para la autenticación de n8n (obligatorio).
- `n8nPassword`: Contraseña para la autenticación de n8n (obligatorio).
- `timeZone`: Zona horaria para n8n (por defecto: "Europe/Rome").
- `fileShareQuotaGB`: Tamaño en GB del archivo compartido para los datos (1-5120, por defecto: 5).

⚠️ **Notas sobre los nombres**:
- El nombre del despliegue (`name`) debe tener al menos 5 caracteres para cumplir con los requisitos de DNS de Azure.
- El prefijo del almacenamiento (`storagePrefix`) se combina con una cadena única para crear un nombre válido para la cuenta de almacenamiento.

## Características

- **Persistencia de Datos**: Todos los datos de n8n (workflows, credenciales, etc.) se guardan de manera persistente en Azure File Share.
- **SSL/TLS Automático**: Caddy gestiona automáticamente los certificados SSL.
- **Autenticación Básica**: Autenticación integrada para proteger el acceso.
- **Optimización de Recursos**: Contenedores configurados con recursos optimizados para un uso típico.
- **Reinicio Automático**: Política de reinicio automático en caso de errores.

## Personalización de los Parámetros

### A través del Portal de Azure
1. Haz clic en el botón "Deploy to Azure" arriba.
2. Completa los parámetros en el formulario que aparece:
   - Selecciona la suscripción y el grupo de recursos.
   - Introduce un nombre válido para el despliegue (mínimo 5 caracteres).
   - **Importante**: Configura el nombre de usuario y la contraseña para el acceso a n8n.
   - Opcionalmente, modifica los otros parámetros (zona horaria, tamaño del archivo compartido).
3. Revisa y crea el despliegue.

### A través de la CLI de Azure

```bash
az deployment group create \
  --resource-group <RESOURCE_GROUP_NAME> \
  --template-file deploy-n8n.json \
  --parameters \
      name=<NAME> \
      n8nUsername=<USERNAME> \
      n8nPassword=<PASSWORD>
```

## Acceso a la Aplicación
Después del despliegue, n8n estará accesible en la siguiente dirección:

```php
https://<name>.<region>.azurecontainer.io
```

## Resolución de Problemas
- Si el contenedor no se inicia, revisa los logs en el portal de Azure.
- Verifica que las credenciales de autenticación sean correctas.
- Asegúrate de que el archivo compartido se haya creado correctamente.
- Para problemas con SSL, revisa los logs del contenedor Caddy.
- Si el despliegue falla por errores en el nombre:
- Asegúrate de que el nombre del despliegue tenga al menos 5 caracteres.
    - Usa solo letras minúsculas, números y guiones.
    - El nombre debe comenzar con una letra.
