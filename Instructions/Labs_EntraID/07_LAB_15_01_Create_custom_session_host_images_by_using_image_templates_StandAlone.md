---
lab:
  title: 'Laboratorio: Creación de imágenes personalizadas de host de sesión mediante plantillas de imagen'
  module: 'Module 1.5: Create and manage session host images'
---

# Laboratorio: Creación de imágenes personalizadas de host de sesión mediante plantillas de imagen
# Manual de laboratorio para estudiantes

## Dependencias de laboratorio

- Una suscripción a Azure que usarás en este laboratorio.
- Una cuenta de usuario de Microsoft Entra con el rol de propietario en la suscripción a Azure que vas a usar en este laboratorio y con los permisos suficientes para unir los dispositivos al inquilino de Microsoft Entra asociado a esa suscripción a Azure.

## Tiempo estimado

90 minutos (el tiempo de espera aproximado para que se complete la compilación de la imagen es de 45 minutos)

## Escenario del laboratorio

Planeas implementar un entorno de Azure Virtual Desktop. Debes usar imágenes de máquina virtual personalizadas al implementar hosts de sesión de Azure Virtual Desktop.

## Objetivos
  
Después de completar este laboratorio, podrás:

- Crear imágenes personalizadas de host de sesión para Azure Virtual Desktop mediante plantillas de imagen

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Creación de imágenes personalizadas de host de sesión mediante plantillas de imagen

Las tareas principales de este ejercicio son las siguientes:

1. Registro de proveedores de recursos necesarios
1. Creación de una identidad administrada asignada por el usuario
1. Creación de un rol personalizado de control de acceso basado en roles de Azure (RBAC)
1. Establecimiento de permisos en los recursos relacionados con el aprovisionamiento de imágenes de host
1. Creación de una definición de imagen y una instancia de Azure Compute Gallery
1. Creación de plantilla de imagen personalizada
1. Compilación de una imagen personalizada
1. Implementación de hosts de sesión mediante una imagen personalizada

> **Nota**: para poder crear una plantilla de imagen personalizada, deberás cumplir varios requisitos previos, entre los que se incluyen:

- Registro de todos los proveedores de recursos necesarios
- Creación de una identidad administrada asignada por el usuario
- Concesión de los permisos necesarios a la identidad administrada asignada por el usuario un rol de control de acceso basado en roles de Azure (RBAC)
- Si piensas distribuir la imagen mediante Azure Compute Gallery, deberás crear su instancia junto con una definición de imagen

#### Tarea 1: Registro de los proveedores de recursos necesarios

1. Si es necesario, desde el equipo de laboratorio, inicia un explorador web, ve al portal de Azure e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que vas a usar en este laboratorio.

    > **Nota**: Usa las credenciales de la cuenta `User1-` que aparecen en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.

1. En Azure Portal, inicia una sesión de PowerShell en Azure Cloud Shell.

    > **Nota**: si se te solicita, en el panel **Introducción**, en la lista desplegable **Suscripción**, selecciona el nombre de la suscripción a Azure que usas en este laboratorio y, a continuación, selecciona **Aplicar**.

1. En la sesión de PowerShell del panel Azure Cloud Shell, ejecute el comando siguiente para registrar el proveedor de recursos **Microsoft.DesktopVirtualization**:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
    Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
    Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance
    ```

    > **Nota**: espera a que se complete el proceso de registro. Esto puede tardar unos cinco minutos.

1. Cierra el panel de Azure Cloud Shell.

#### Tarea 2: Creación de una identidad administrada asignada por el usuario

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Identidades administradas**.
1. En la página **Identidades administradas**, selecciona **+ Crear**.
1. En la pestaña **Datos básicos** de la página **Crear identidad administrada asignada por el usuario**, especifica la siguiente configuración y, a continuación, selecciona **Revisar y crear**:

    > **Nota**: al establecer el valor de **Nombre**, cambia a la pestaña Recursos de la ventana de sesión del laboratorio e identifica la cadena de caracteres entre *User1-* y el carácter *@*. Usa esta cadena para reemplazar el marcador de posición *aleatorio*.

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-15a-RG**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Nombre|**az140**-*random*-**uami**|

1. En la pestaña **Revisar y crear**, selecciona **Crear**.

    >**Nota**: no esperes a que se complete el aprovisionamiento de la identidad administrada asignada por el usuario. Esto tardará unos segundos.

#### Tarea 3: Creación de un rol de control de acceso basado en roles de Azure (RBAC)

>**Nota**: el rol de control de acceso basado en roles de Azure (RBAC) personalizado se usará para asignar los permisos adecuados a la identidad administrada asignada por el usuario que creaste en la tarea anterior.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, inicia una sesión de PowerShell en Azure Cloud Shell.

1. En la sesión de PowerShell del panel Azure Cloud Shell, ejecuta el siguiente comando para identificar el valor de la propiedad **Id.** de la suscripción a Azure que usas en este laboratorio y almacenarlo en la variable **$subscriptionId**:

    ```powershell
    $subscriptionId = (Get-AzSubscription).Id
    ```

1. Ejecuta el siguiente comando para crear la definición de rol del nuevo rol personalizado, incluido su valor de ámbito asignable y almacenarlo en la variable **$jsonContent** (asegúrate de reemplazar el marcador de posición *aleatorio* por la misma cadena que identificaste en la tarea anterior):

    ```powershell
    $jsonContent = @"
    {
      "Name": "Desktop Virtualization Image Creator (random)",
      "IsCustom": true,
      "Description": "Create custom image templates for Azure Virtual Desktop images.",
      "Actions": [
        "Microsoft.Compute/galleries/read",
        "Microsoft.Compute/galleries/images/read",
        "Microsoft.Compute/galleries/images/versions/read",
        "Microsoft.Compute/galleries/images/versions/write",
        "Microsoft.Compute/images/write",
        "Microsoft.Compute/images/read",
        "Microsoft.Compute/images/delete"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/$subscriptionId",
        "/subscriptions/$subscriptionId/resourceGroups/az140-15b-RG"
      ]
    }
    "@
    ```

1. Ejecuta el comando siguiente para almacenar el contenido de la variable **$jsonContent** en un archivo denominado **CustomRole.json**:

    ```powershell
    $jsonContent | Out-File -FilePath 'CustomRole.json'
    ```

1. Ejecuta el siguiente comando para crear el nuevo rol personalizado:

    ```powershell
    New-AzRoleDefinition -InputFile ./CustomRole.json
    ```

1. Cierra el panel de Azure Cloud Shell.

#### Tarea 4: Establecimiento de permisos en los recursos relacionados con el aprovisionamiento de imágenes de host

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Grupos de recursos** y, en la página **Grupos de recursos**, selecciona **+ Crear**.
1. En la pestaña **Datos básicos** de la página **Crear un grupo de recursos**, especifica la siguiente configuración y, a continuación, selecciona **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-15b-RG**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|

1. En la pestaña **Revisar y crear**, selecciona **Crear**.
1. Actualiza la página **Grupos de recursos** y, en la lista de grupos de recursos, selecciona **az140-15b-RG**.
1. En la página **az140-15b-RG**, en el menú de navegación vertical, selecciona **Control de acceso (IAM)**.
1. En la página **az140-15b-RG\|Control de acceso (IAM)**, selecciona **+ Agregar** y, después, en el menú desplegable, selecciona **Agregar asignación de roles**.
1. En la pestaña **Rol** de la página **Agregar asignación de roles**, asegúrate de que está seleccionada la pestaña **Roles de función de trabajo**; en el cuadro de texto de búsqueda, escribe **Generador de imágenes de Desktop Virtualization** (*aleatorio*), en la lista de resultados, selecciona **Generador de imágenes de Desktop Virtualization** (*aleatorio*) y, a continuación, selecciona **Siguiente**.

    >**Nota**: asegúrate de reemplazar el marcador de posición *aleatorio* por la misma cadena que usaste al definir el nuevo rol RBAC personalizado.

1. En la pestaña **Miembros** de la página **Agregar asignación de roles**, selecciona la opción **Identidad administrada**, haz clic en **+ Seleccionar miembros**, en el panel **Seleccionar identidades administradas**, en la lista desplegable **Identidad administrada**, selecciona **Identidad administrada asignada por el usuario**, en la lista de identidades administradas asignadas por el usuario, selecciona **az140**-*random*-** uami** (donde el marcador de posición *random* representa la misma cadena que usaste al definir el nuevo rol RBAC personalizado) y, a continuación, haz clic en **Seleccionar**.
1. De nuevo en la pestaña **Miembros** de la página **Agregar asignación de roles**, selecciona **Revisar y asignar**.
1. En la pestaña **Revisar y asignar**, selecciona **Revisar y asignar**. 

#### Tarea 5: Creación de una instancia y una definición de imagen de Azure Compute Gallery

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Galerías de Azure Compute** y, en la página **Galerías de Azure Compute**, selecciona **+ Crear**.
1. En la pestaña **Datos básicos** de la página **Crear Azure Compute Gallery**, especifica la siguiente configuración y, a continuación, selecciona **Siguiente: Método de uso compartido**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-15b-RG**|
    |Nombre|**az14015computegallery**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|

1. En la pestaña **Uso compartido** de la página **Crear Azure Compute Gallery**, deja la opción predeterminada **Control de acceso basado en roles (RBAC)** seleccionada y, a continuación, selecciona **Revisar y crear**.
1. En la pestaña **Revisar y crear**, selecciona **Crear**.

    >**Nota**: espera a que se complete el proceso de aprovisionamiento. Debería tardar menos de un minuto.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Galerías de Azure Compute** y, en la página **Galerías de Azure Compute**, selecciona **az14015computegallery**. 
1. En la página **az14015computegallery**, selecciona **+ Agregar** y, en el menú desplegable, selecciona **+ Definición de imagen de VM**. 
1. En la pestaña **Datos básicos** de la página **Crear definición de imagen de VM**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada) y selecciona **Siguiente: Versión**:

    |Configuración|Valor|
    |---|---|
    |Region|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Nombre de definición de la imagen de VM|**az14015imagedefinition**|
    |Tipo de SO|**Windows**|
    |Tipo de seguridad|**Inicio seguro admitido**|
    |Estado del SO|**Generalizado**|
    |Publicador|**MicrosoftWindowsDesktop**|
    |Oferta|**Windows-11**|
    |SKU|**win11-23h2-avd-m365**|

    > **Nota**: la generación de máquinas virtuales se establece automáticamente en Gen2, ya que las máquinas virtuales Gen 1 no se admiten con el tipo de seguridad de confianza y confidencial.

1. En la pestaña **Versión** de la página **Crear definición de imagen de VM**, deja la configuración sin cambios y selecciona **Siguiente: Opciones de publicación**.

    > **Nota**: no debes crear la versión de la imagen de VM en esta fase. Azure Virtual Desktop lo hará.

1. En la pestaña **Opciones de publicación** de la página **Crear definición de imagen de VM**, deja la configuración sin cambios y selecciona **Revisar y crear**.
1. En la pestaña **Revisar y crear** de la página **Crear definición de imagen de VM**, selecciona **Crear**.

    > **Nota**: espera a que se complete el proceso de aprovisionamiento. Esto suele tardar menos de un minuto.

#### Tarea 6: Creación de plantilla de imagen personalizada

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Plantillas de imagen personalizadas** y, en la página **Azure Virtual Desktop \| Plantillas de imagen personalizadas**, selecciona **+ Agregar plantilla de imagen personalizada**. 
1. En la pestaña **Datos básicos** de la página **Crear plantilla de imagen personalizada**, especifica la siguiente configuración y selecciona **Siguiente**:

    > **Nota**: al establecer la propiedad **Identidad administrada**, asegúrate de reemplazar el marcador de posición *aleatorio* por la misma cadena que identificaste anteriormente en este ejercicio.

    |Configuración|Valor|
    |---|---|
    |Nombre de plantilla|**az140-15b-imagetemplate**|
    |Importación desde una plantilla existente|**No**|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-15b-RG**|
    |Ubicación|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Identidad administrada|**az140**-*random*-**uami**|

1. En la pestaña **Imagen de origen** de la página **Crear plantilla de imagen personalizada**, especifica la siguiente configuración y selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Tipo de origen|**Imagen de plataforma (Marketplace)**|
    |Seleccionar imagen|**Windows 11 Enterprise para sesiones múltiples, versión 23H2 + Aplicaciones de Microsoft 365**|

1. En la pestaña **Destinos de distribución** de la página **Crear plantilla de imagen de VM**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada) y selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Azure Compute Gallery|habilitado|
    |Nombre de la galería|**az14015computegallery**|
    |Definición de imagen de la galería|**az14015imagedefinition**|
    |Versión de imagen de la galería|**1.0.0**|
    |Nombre de salida de la ejecución|**az140-15-image-1.0.0**|
    |Regiones de replicación|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Excluir de la versión más reciente|**No**|
    |Tipo de cuenta de almacenamiento|**Standard_LRS**|

    > **Nota**: puedes usar la propiedad **Regiones de replicación** para dar cabida a las compilaciones de varias regiones. Si se establece **Excluir de la versión más reciente** a **Sí**, se impediría que esta versión de imagen se use cuando se especifica **la versión más reciente** como la versión del elemento **ImageReference** durante la creación de la máquina virtual.

1. En la pestaña **Propiedades de compilación** de la página **Crear plantilla de imagen personalizada**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada) y selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Tiempo de expiración de compilación|**120**|
    |Tamaño de la máquina virtual de compilación|**Standard_DC2s_v3**|
    |Tamaño del disco del sistema operativo (GB)|**127**|
    |Grupo de almacenamiento provisional|**az140-15c-RG**|
    |VNet|Deja en no establecido|

    > **Nota**: El **grupo de almacenamiento provisional** es el grupo de recursos que se usa para almacenar provisionalmente los recursos para compilar la imagen y almacenar los registros. Si no proporcionas un nombre, se generará automáticamente. Si no se establece el nombre de **VNet**, se crea uno temporal, junto con una dirección IP pública para la máquina virtual que se usa para crear la compilación.

    > **Importante**: asegúrate de que tienes el número suficiente de vCPU disponibles para el tamaño de la máquina virtual de compilación que especificaste. Si no es así, elige un tamaño diferente o solicita un aumento de cuota.

1. En la pestaña **Personalización** de la página **Crear plantilla de imagen personalizada**, selecciona **+ Agregar script integrado**. 
1. En el panel **Seleccionar scripts integrados**, revisa las opciones disponibles agrupadas en los scripts específicos del sistema operativo, los scripts de Azure Virtual Desktop, los scripts de conexión de aplicaciones MSIX, los scripts de aplicación y scripts relacionados con actualizaciones de Windows y, a continuación, selecciona las siguientes entradas:

   - **Redirección de zona horaria**: permite al cliente usar su zona horaria dentro de la sesión en los hosts de sesión
   - **Deshabilitar sensor de almacenamiento**: impide que el sensor de almacenamiento afecte negativamente a los hosts de sesión al detectar falsamente las condiciones de espacio disponible bajo en el disco.
   - **Habilitar de la protección de captura de pantalla** con **bloquear captura de pantalla en el cliente y el servidor**: bloquea u oculta el contenido remoto en las capturas y el uso compartido de pantalla

1. En el panel **Seleccionar scripts integrados**, selecciona **Guardar**.

    > **Nota**: tienes la opción de agregar tus propios scripts. Para obtener ejemplos, considera la posibilidad de hacer referencia a los scripts integrados, como [Redirección de zona horaria](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/TimezoneRedirection.ps1), [Deshabilitar sensor de almacenamiento](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/DisableStorageSense.ps1) o [Habilitar protección de captura de pantalla](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/ScreenCaptureProtection.ps1).

1. De nuevo en la pestaña **Personalización** de la página **Crear plantilla de imagen personalizada**, selecciona **Siguiente**.
1. En la pestaña **Etiquetas** de la página **Crear plantilla de imagen personalizada**, selecciona **Siguiente**.
1. En la pestaña **Revisar y crear** de la página **Crear plantilla de imagen personalizada**, selecciona **Crear**.

    > **Nota**: espera a que se cree la plantilla. Esto puede tardar unos minutos. Actualiza la página **Azure Virtual Desktop \| Plantillas de imagen personalizadas** para revisar el estado de la plantilla.

#### Tarea 7: Creación de una imagen personalizada

> **Nota**: las tareas restantes de este laboratorio son opcionales, ya que implican un tiempo de espera bastante largo. 

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, en la página **Azure Virtual Desktop \| Plantilla de imagen personalizada**, selecciona **az140-15b-imagetemplate**.
1. En la página **az140-15b-imagetemplate**, selecciona **Iniciar compilación**.

    > **Nota:** espera a que se cree la compilación. El tiempo real para completar el proceso de compilación puede variar, pero con la configuración proporcionada en las instrucciones del laboratorio, debe completarse en 45 minutos. Actualiza la página cada pocos minutos y supervisa el valor de **Estado de ejecución de compilación** en la sección **Essentials** de la página **az140-15b-imagetemplate**. 

    > **Nota**: el estado de ejecución de compilación debe cambiar en algún momento de **Ejecución - Compilación** a **Ejecución - Distribución** y, por último, a **Correcto**.

    > **Nota**: mientras esperas a que se complete la compilación, revisa el contenido del grupo de recursos de almacenamiento provisional **az140-15c-RG**, donde los recursos de compilación, incluida la máquina virtual de compilación, la red virtual, el grupo de seguridad de red, el almacén de claves, la instantánea, la instancia de contenedor y la cuenta de almacenamiento se aprovisionan automáticamente. 

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Grupos de recursos** y, en la página **Grupos de recursos**, selecciona **az140-15c-RG**.
1. En la página **az140-15c-RG**, en la sección **Recursos**, anota los recursos provisionados automáticamente.
1. Vuelve a la página **az140-15b-imagetemplate** y supervisa el progreso de la compilación. 

    > **Nota**: como alternativa, puedes usar el **registro de actividad** para realizar el seguimiento de la finalización del proceso de compilación. La acción en la que debes centrarte es **Ejecutar una plantilla de imagen de VM para generar su salida**. Su estado debe cambiar en algún momento de **Aceptado** a **Correcto**.

1. Una vez completada la compilación, en el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Galerías de Azure Compute** y, en la página **Galerías de Azure Compute**, selecciona **az14015computegallery**. 
1. En **az14015computegallery**, en la pestaña **Definiciones**, selecciona **az14015imagedefinition**.
1. En la página **az14015imagedefinition**, en la pestaña **Versiones**, revisa la información sobre la imagen **1.0.0 (versión más reciente)**.

#### Tarea 8: Implementación de hosts de sesión mediante una imagen personalizada

> **Nota**: opcionalmente, considera la posibilidad de recorrer paso a paso las fases iniciales de la implementación de los hosts de sesión de Azure Virtual Desktop mediante la imagen personalizada que creaste. 

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Redes virtuales** y, en la página **Redes virtuales**, selecciona **Crear +**.
1. En la pestaña **Datos básicos** de la página **Crear red virtual**, especifica la siguiente configuración y selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-15d-RG**.|
    |Nombre de la red virtual|**az140-vnet15d**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|

1. En la pestaña **Seguridad**, acepta la configuración predeterminada y selecciona **Siguiente**.
1. En la pestaña **Direcciones IP**, especifica la siguiente configuración:

    |Configuración|Valor|
    |---|---|
    |Espacio de direcciones IP|**10.30.0.0/16**|

1. Selecciona el icono de edición (lápiz) situado junto a la entrada de subred **predeterminada**, en el panel **Editar**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada) y selecciona **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|**hp1-Subnet**|
    |Dirección inicial|**10.30.1.0**|
    |Habilitar subred privada (sin acceso de salida predeterminado)|Deshabilitado|

1. Vuelve a la pestaña **Direcciones IP**, selecciona **Revisar y crear** y, a continuación, selecciona **Crear**.

    > **Nota**: espera a que se complete el proceso de aprovisionamiento. Esto suele tardar menos de un minuto.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos de hosts** y, en la página **Azure Virtual Desktop \| Grupos de hosts**, selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la página **Crear un grupo de hosts**, especifica la siguiente configuración y selecciona **Siguiente: Hosts de sesión >** (deja los otros valores con su configuración predeterminada):

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-15d-RG**|
    |Nombre del grupo de hosts|**az140-15-hp1**|
    |Ubicación|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Entorno de validación|**No**|
    |Tipo de grupo de aplicaciones preferido|**Dispositivo de escritorio**|
    |Tipo de grupo de hosts|**Agrupado**|
    |Crear configuración del host de sesión|**No**|
    |Algoritmo de equilibrio de carga|**Equilibrio de carga en amplitud**|

    > **Nota**: al usar el algoritmo de equilibrio de carga en amplitud, el parámetro límite máximo de sesión es opcional.

1. En la pestaña **Hosts de sesión** de la página **Crear un grupo de hosts**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada):

    > **Nota**: al establecer el valor de **prefijo de nombre**, cambia a la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio e identifica la cadena de caracteres entre *User1-* y el carácter *@*. Usa esta cadena para reemplazar el marcador de posición *aleatorio*.

    |Configuración|Valor|
    |---|---|
    |Agregar máquinas virtuales|**Sí**|
    |Grupo de recursos|**El valor predeterminado es el mismo que el del grupo de hosts**|
    |Prefijo de nombre|**sh0**_random_|
    |Tipo de máquina virtual|**Máquina virtual de Azure**|
    |Ubicación de la máquina virtual|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
    |Tipo de seguridad|**Máquina virtual de inicio seguro**|

1. En la pestaña **Máquinas virtuales** de la página **Crear un grupo de hosts**, debajo de la lista desplegable **Imagen**, selecciona **Ver todas las imágenes**.
1. En la página **Seleccionar una imagen**, selecciona **Imágenes compartidas** y, en la lista de imágenes, selecciona **az14015imagedefinition**. 
1. De nuevo en la pestaña **Máquinas virtuales** de la página **Crear un grupo de hosts**, especifica la siguiente configuración y selecciona **Siguiente: Área de trabajo >** (deja los otros valores con su configuración predeterminada):

    |Configuración|Valor|
    |---|---|
    |Tamaño de la máquina virtual|**Standard DC2s_v3**|
    |Número de máquinas virtuales|**1**|
    |Tipo de disco del sistema operativo|**SSD estándar**|
    |Tamaño de disco del SO|**Tamaño predeterminado**|
    |Diagnósticos de arranque|**Habilitar con la cuenta de almacenamiento administrada (recomendado)**|
    |Red virtual|az140-vnet15d|
    |Subred|**hp1-Subnet**|
    |Grupo de seguridad de red|**Basic**|
    |Puertos de entrada públicos|**No**|
    |Selecciona el directorio al que quieras unirte|**Microsoft Entra ID**|
    |Inscripción de una máquina virtual con Intune|**No**|
    |Nombre de usuario|**Estudiante**|
    |Contraseña|Cualquier cadena de caracteres suficientemente compleja que se usará como contraseña para la cuenta predefinida de administrador|
    |Confirmar contraseña|La misma cadena de caracteres que especificaste anteriormente|

    > **Nota**: la contraseña debe tener al menos 12 caracteres de longitud y consta de una combinación de caracteres en minúsculas, mayúsculas, dígitos y caracteres especiales. Para obtener más información, consulta la información sobre [los requisitos de contraseña al crear una VM de Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. En la pestaña **Área de trabajo** de la página **Crear un grupo de hosts**, confirma la siguiente configuración y selecciona **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Registro de un grupo de aplicaciones de escritorio|**No**|

1. En la pestaña **Revisar y crear** de la página **Crear un grupo de hosts**, selecciona **Crear**.

    > **Nota**: espera a que la implementación se complete. Esto puede llevar unos 10-15 minutos.
