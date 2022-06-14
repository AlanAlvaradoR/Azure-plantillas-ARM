# Azure-plantillas-ARM

- Crear tu propia plantilla ARM
- Obtener una plantilla ARM ya hecha
  - Opción 1: Descargarla de un recurso existente en Azure Portal
  - Opción 2: Descargar la plantilla de todo un grupo de recursos
  - Opción 3: Obtenerla de la comunidad

## Crea tu propia plantilla ARM

1. Abrir editor de código o IDE. En este caso se usará [VS Code](https://code.visualstudio.com/) el cual debe tener instalada la extensión [Azure Resource Manager (ARM) Tools](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
2. Crea un archivo nuevo donde se pondrá la información de la plantilla llamado `azuredeploy.json`
3. Copia y pega lo siguiente dentro del nuevo archivo
```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": []
}
```

- `$schema`: Especifica la ubicación del archivo de esquema JSON. El archivo de esquema describe las propiedades que están disponibles dentro de una plantilla
- `contentVersion`: Especifica la versión de la plantilla (por ejemplo, 1.0.0.0)
- `resources`: Contiene los recursos que desea implementar o actualizar. Actualmente está vacía, pero agregará recursos más adelante

4. Abre un CMD dentro de la carpeta que contiene el archivo recién creado y ejecuta el siguiente comando en el [CLI de Azure](https://docs.microsoft.com/es-es/cli/azure/install-azure-cli), en este caso el nombre de la plantilla será "mi-primera-plantilla", dentro del grupo de recursos "sesion7" y con el archivo JSON llamado "azuredeploy.json"
```
az deployment group create --name mi-primera-plantilla --resource-group sesion7 --template-file azuredeploy.json
```

Si te aparece lo siguiente quiere decir que la plantilla fue implementada con éxito:
```
"ProvisioningState": "Succeeded",
```

5. Ve a portal.azure.com y dirigete a tu grupo de recursos. Después da clic en Implementaciones o *Deplyments*

![Captura portal Azure](/res/images/img-arm5.jpg)

6. Aquí encontrarás la plantilla que acabas de crear junto con todas las implementaciones que se han hecho desde Azure Portal, Cloud Shell o Azure CLI

![Captura portal Azure](/res/images/img-arm6.jpg)

Si el nombre de tu plantilla aparece como correcta quiere decir que se implementó bien. Dentro ya puedes tener más información de su ejecución

7. Se va a agregar un recurso a la plantilla, será una cuenta de almacenamiento llamada "micuentadealmacenamiento". 
```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "micuentadealmacenamiento",
      "location": "eastus",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true
      }
    }
  ]
}
```

Cada recurso implementado tiene al menos las siguientes tres propiedades:

- `type`: tipo de recurso. Este valor es una combinación del espacio de nombres del proveedor de recursos y el tipo de recurso, como `Microsoft.Storage/storageAccounts`
- `apiVersion`: Versión de la API de REST que debe usar para crear el recurso. Cada proveedor de recursos publica sus propias versiones de API, por lo que este valor es específico del tipo
- `name`: Nombre del recurso

En este caso se agregaron más propiedades, como lo son:

- `location`: región de Azure donde se implementará el recurso
- `sku`: nivel de producto o servicio con el que se creará, en este caso Standard.
- `kind`: tipo de almacenamiento usado para el recurso. Este recurso puede no usarse para todos los casos.
- `properties`: Se establecen propiedades especificas para este recurso. En este caso se le impone al recurso el solo permitir solicitudes HTTPS de salida y entrada.

8. Repite los pasos 4, 5 y 6 de nuevo. 

**Ahora, la plantilla que tenemos le vamos a agregar un parametro.**

9. Vas a agregar lo siguiente en la plantilla. Esto debe estar en el primer nivel, es decir, al mismo nivel que `contentVersion`


```JSON
"parameters": {
    "storageName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24
    }
  },
```

Dentro de `resources` debes agregar lo siguiente:
```JSON
"name": "[parameters('storageName')]",
```

De tal forma que el código completo de la plantilla sea el siguiente:

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[parameters('storageName')]",
      "location": "eastus",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true
      }
    }
  ]
}
```

El parametro que estamos definiendo se llama `storageName` y es un *string* o cadena de caracteres que se ingresa la momento de invocar el template con Azure CLI.

Este parametro permite ser usado a lo largo de toda la plantilla y podríamos poner un SKU común para todos los recursos, un nombre similar, la localización o cualquier cosa que queramos y sea compatible con el recurso y con la sintaxis de las plantillas.

10. Para ejecutar este nueva plantilla necesitas ejecutar el siguiente comando sustituyendo los valores en las palabras en mayusculas:


```
az deployment group create \
  --name NOMBRE_PLANTILLA \
  --resource-group NOMBRE_GRUPO_RECURSOS \
  --template-file DIRECCION_ARCHIVO_PLANTILLA \
  --parameters storageName=VALOR_storageName
```

**Nota**: Lo que pongas en vez de VALOR_storageName será el valor que tenga esta variable dentro de toda la plantilla. Estos parametros se pueden considerar como constantes en un lenguaje de programación.

**Y así como podemos poner parametros o constantes, también podemos colocar variables**

11. Agregarás a tu plantilla lo siguiente:

```JSON
"storagePrefix": {
      "type": "string",
      "minLength": 3,
      "maxLength": 11
    },
"variables": {
    "uniqueStorageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]"
  },
"name": "[variables('uniqueStorageName')]",
```

Dando como resultado lo siguiente:

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "type": "string",
      "minLength": 3,
      "maxLength": 11
    }
  },
  "variables": {
    "uniqueStorageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[variables('uniqueStorageName')]",
      "location": "eastus",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true
      }
    }
  ]
}
```

En la sección variables se une el valor que introduces en el CLI como storagePrefix con un *string* aleatorio, esto con el fin de evitar que se repitan los nombres en los mismos recursos y pueda derivar en un error de implementación.

12. Ejecutas la plantilla de ARM con el siguiente comando de Azure CLI

```
az deployment group create \
  --name NOMBRE_PLANTILLA \
  --resource-group NOMBRE_GRUPO_RECURSOS \
  --template-file DIRECCION_ARCHIVO_PLANTILLA \
  --parameters storagePrefix=VALOR_storagePrefix
```

Después de la ejecución exitosa de este comando te debe de crear un recurso similar a este:

![Captura portal Azure](/res/images/img-arm7.jpg)

Si quieres aprender más sobre la creación de las plantillas consulta la siguiente documentación:
- [Documentación de las plantillas de Resource Manager
](https://docs.microsoft.com/es-mx/azure/azure-resource-manager/templates/)
- [Buenas practicas para plantillas ARM](https://github.com/Azure/azure-quickstart-templates/blob/master/)

## Obtener una plantilla de ARM ya hecha

### Opción 1: Descargarla de un recurso existente en Azure Portal

1. Dirigete a portal.azure.com
2. Una vez dentro, explora el detalle de un recurso haciendo clic desde la página principal o desde la sección **Todos los recursos**
3. Una vez dentro del recurso, desliza el menu lateral izquierdo hasta el fondo y encontrarás la opción **Exportar plantilla**

![Exportar plantilla](https://github.com/AlanAlvaradoR/Azure-plantillas-ARM/blob/main/imagenes/1.PNG)

4. Encontrarás una interfaz similar a esta:

![Exportación de plantilla](https://github.com/AlanAlvaradoR/Azure-plantillas-ARM/blob/main/imagenes/2.PNG)

Dentro de esta interfaz podrás:

- **Descargar** la plantilla que resultará en dos archivos JSON editablesque contienen:
  - La o las plantillas
  - Los parametros usados en todas las plantillas. Por ejemplo, una dirección de serivdor, una dirección IP o un grupo de recursos único
- **Implementar** la plantilla para hacer una replica del recurso en el que te encuentras
- **Guardar en la biblioteca**. Donde podrás encontrar las plantillas que más uses o que necesites para después.

**Nota**: No puedes editar una plantilla desde el Azure Portal, para ello tendrías que descargarlas. Además, puedes bajarla o implementarla con o sin los parametros.

### Opción 2: Descargar la plantilla de todo un grupo de recursos

1. Dirigete a portal.azure.com
2. Una vez dentro, explora el detalle de un recurso haciendo clic desde la página principal o desde la sección **Todos los recursos**. Si no te aparece este botón, puedes buscarlo utilizando la barra de busqueda superior.
3. Selecciona un grupo de recursos
4. Una vez dentro del grupo de recursos, desliza el menu lateral izquierdo hasta el fondo y encontrarás la opción **Exportar plantilla**
5. Las opciones son las mismas que al descargar la plantilla de un solo recurso solo con la diferencia que aquí exportas la implementación de todos los recursos dentro del grupo.

### Opción 3: Obtenerla de la comunidad

Estas plantillas son creadas por personas de la comunidad de Azure de todo el mundo y todas están validadas para su uso.

1. Ve al repositorio de [Quickstart Templates](https://github.com/Azure/azure-quickstart-templates)
2. Escoge una plantilla y al hacer clic en ella te aparecerá la siguiente interfaz:

![Github](https://github.com/AlanAlvaradoR/Azure-plantillas-ARM/blob/main/imagenes/3.PNG)

Donde puedes:

- Ver el código JSON de la plantilla en el archivo `azuredeploy.json`
- Implementarla en tu cuenta de Azure con un solo clic desde el botón Deploy to Azure. Esto te abrirá una ventana de portal.azure.com con todas las configuraciones de la plantilla donde tu puedes modificar los parametros y ordenar la ejecución manualmente
- Implementarla en un ambiente de Azure Gov si esta se encuentra validada para ello
- Ver las pruebas que ha pasado por medio de *badges*
- Verla en un diagrama
