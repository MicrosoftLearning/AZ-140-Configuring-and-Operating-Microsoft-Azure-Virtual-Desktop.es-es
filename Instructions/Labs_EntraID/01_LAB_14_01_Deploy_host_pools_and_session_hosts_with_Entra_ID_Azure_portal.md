---
lab:
  title: 'Laboratorio: Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)'
  module: 'Module 1.4: Implement host pools and session hosts'
---

# Laboratorio: Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)
# Manual de laboratorio para estudiantes

## Dependencias de laboratorio

- Una suscripción a Azure que usarás en este laboratorio.
- Una cuenta de usuario de Microsoft Entra con el rol de propietario en la suscripción a Azure que vas a usar en este laboratorio y con los permisos suficientes para unir los dispositivos al inquilino de Microsoft Entra asociado a esa suscripción a Azure.

## Tiempo estimado

60 minutos

## Escenario del laboratorio

Tienes una suscripción a Microsoft Azure. Debes implementar el entorno de Azure Virtual Desktop que use hosts de sesión unidos a Microsoft Entra.

## Objetivos
  
Después de completar este laboratorio, podrás:

- Implementar hosts de sesión unidos a Microsoft Entra en Azure Virtual Desktop

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Implementación de un entorno de Azure Virtual Desktop mediante hosts de sesión unidos a Microsoft Entra
  
Las tareas principales de este ejercicio son las siguientes:

1. Preparación de la suscripción a Azure para la implementación de un grupo de hosts de Azure Virtual Desktop
1. Implementación de un grupo de hosts de Azure Virtual Desktop
1. Creación de un grupo de aplicaciones de Azure Virtual Desktop
1. Creación de un área de trabajo de Azure Virtual Desktop
1. Concesión de acceso al grupo de hosts de Azure Virtual Desktop

#### Tarea 1: Preparación de la suscripción a Azure para la implementación de un grupo de hosts de Azure Virtual Desktop

1. En el equipo de laboratorio, inicia un explorador web, ve a Azure Portal en [https://portal.azure.com](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol de propietario en la suscripción que usarás en este laboratorio.

    > **Nota**: Usa las credenciales de la cuenta `User1-` que aparecen en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.

1. En Azure Portal, inicia una sesión de PowerShell en Azure Cloud Shell.

    > **Nota**: si se te solicita, en el panel **Introducción**, en la lista desplegable **Suscripción**, selecciona el nombre de la suscripción a Azure que usas en este laboratorio y, a continuación, selecciona **Aplicar**.

1. En la sesión de PowerShell del panel Azure Cloud Shell, ejecute el comando siguiente para registrar el proveedor de recursos **Microsoft.DesktopVirtualization**:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    ```

    > **Nota**: espera a que se complete el proceso de registro. Esto puede tardar un par de minutos.

1. Cierra el panel de Cloud Shell.
1. En el explorador web donde se muestra Azure Portal, busca y selecciona **Redes virtuales** y, en la página **Redes virtuales**, selecciona **Crear +**.
1. En la pestaña **Datos básicos** de la página **Crear red virtual**, especifica la siguiente configuración y selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-11e-RG**|
    |Nombre de la red virtual|**az140-vnet11e**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|

1. En la pestaña **Seguridad**, acepta la configuración predeterminada y selecciona **Siguiente**.
1. En la pestaña **Direcciones IP**, aplica la siguiente configuración (modifica el valor predeterminado si es necesario):

    |Configuración|Valor|
    |---|---|
    |Espacio de direcciones IP|**10.20.0.0/16**|

1. Selecciona el icono de edición (lápiz) situado junto a la entrada de subred **predeterminada**, en el panel **Editar**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada) y selecciona **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|**hp1-Subnet**|
    |Dirección inicial|**10.20.1.0**|
    |Habilitar subred privada (sin acceso de salida predeterminado)|Deshabilitado|

1. De nuevo en la pestaña **Direcciones IP**, selecciona **Revisar y crear** y, a continuación, en la pestaña **Revisar y crear**, selecciona **Crear**.

    > **Nota**: no esperes a que se complete el proceso de aprovisionamiento. Esto suele tardar menos de un minuto.

1. En el explorador web que muestra Azure Portal, busca y selecciona **Microsoft Entra ID**.
1. En la página **Información general** del inquilino de Microsoft Entra asociado a tu suscripción, en la sección **Administrar** del menú de navegación vertical, selecciona **Usuarios**.
1. En la página **Usuarios**, en el cuadro de texto **Buscar**, escribe el nombre de la cuenta `User1-` que aparece en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.
1. En la lista de resultados de la búsqueda, selecciona la entrada de la cuenta de usuario con el nombre coincidente.
1. En la página que muestra las prioridades de la cuenta de usuario, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos**.
1. En la página **Grupos**, registra el nombre del grupo a partir del prefijo **AVD-DAG** (lo necesitarás más adelante en este laboratorio).
1. Vuelve a la página **Usuarios**, en el cuadro de texto **Buscar**, escribe el nombre de la cuenta `User2-` que aparece en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.
1. En la lista de resultados de la búsqueda, selecciona la entrada de la cuenta de usuario con el nombre coincidente.
1. En la página que muestra las prioridades de la cuenta de usuario, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos**.
1. En la página **Grupos**, registra el nombre del grupo a partir del prefijo **AVD-RemoteApp** (lo necesitarás más adelante en este laboratorio).

#### Tarea 2: Implementación de un grupo de hosts de Azure Virtual Desktop

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos de hosts** y, en la página **Azure Virtual Desktop \| Grupos de hosts**, selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la página **Crear un grupo de hosts**, especifica la siguiente configuración y selecciona **Siguiente: Hosts de sesión >** (deja los otros valores con su configuración predeterminada):

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-21e-RG**|
    |Nombre del grupo de hosts|**az140-21-hp1**|
    |Location|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Entorno de validación|**No**|
    |Tipo de grupo de aplicaciones preferido|**Dispositivo de escritorio**|
    |Tipo de grupo de hosts|**Agrupado**|
    |Crear configuración del host de sesión|**No**|
    |Algoritmo de equilibrio de carga|**Equilibrio de carga en amplitud**|

    > **Nota**: al usar el algoritmo de equilibrio de carga en amplitud, el parámetro límite máximo de sesión es opcional.

1. En la pestaña **Hosts de sesión** de la página **Crear un grupo de hosts**, especifica la siguiente configuración y selecciona **Siguiente: Área de trabajo >** (deja los otros valores con su configuración predeterminada):

    > **Nota**: al establecer el valor de **prefijo de nombre**, cambia a la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio e identifica la cadena de caracteres entre *User1-* y el carácter *@*. Usa esta cadena para reemplazar el marcador de posición *aleatorio*.

    |Configuración|Valor|
    |---|---|
    |Agregar máquinas virtuales|**Sí**|
    |Grupo de recursos|**El valor predeterminado es el mismo que el del grupo de hosts**|
    |Prefijo de nombre|**sh**-*random*|
    |Tipo de máquina virtual|**Máquina virtual de Azure**|
    |Ubicación de la máquina virtual|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
    |Tipo de seguridad|**Máquina virtual de inicio seguro**|
    |Imagen|**Windows 11 Enterprise para sesiones múltiples, versión 23H2 + Aplicaciones de Microsoft 365**|
    |Tamaño de la máquina virtual|**Standard DC2s_v3**|
    |Número de máquinas virtuales|**2**|
    |Tipo de disco del sistema operativo|**SSD estándar**|
    |Tamaño de disco del SO|**Tamaño predeterminado (128GiB)**|
    |Diagnósticos de arranque|**Habilitar con la cuenta de almacenamiento administrada (recomendado)**|
    |Red virtual|**az140-vnet11e**|
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

    > **Nota**: espera a que la implementación se complete. Esto puede tardar unos 20 minutos.

#### Tarea 3: Creación de un grupo de aplicaciones de Azure Virtual Desktop

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, selecciona **Grupos de aplicaciones**.
1. En la página **Azure Virtual Desktop \| Grupos de aplicaciones**, fíjate en el grupo de aplicaciones de escritorio existente y generado automáticamente **az140-21-hp1-DAG** y selecciónalo. 
1. En la página **az140-21-hp1-DAG**, en la sección **Administrar** del menú de navegación vertical, selecciona **Asignaciones**.
1. En la página **az140-21-hp1-DAG \| Asignaciones**, selecciona **+ Agregar**.
1. En la página **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, selecciona **Grupos**, en el cuadro de búsqueda, escribe el nombre completo del grupo **AVD-DAG** que identificaste en la primera tarea de este ejercicio, activa la casilla situada junto al nombre del grupo y haz clic en **Seleccionar**.
1. Vuelve a la página **Azure Virtual Desktop \| Grupos de aplicaciones** y selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la página **Crear un grupo de aplicaciones**, especifica la siguiente configuración y selecciona **Siguiente: Aplicaciones >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-21e-RG**|
    |Grupo de hosts|**az140-21-hp1**|
    |Tipo de grupo de aplicaciones|**Aplicación remota**|
    |Nombre del grupo de aplicaciones|**az140-21-hp1-Office365-RAG**|

1. En la pestaña **Aplicaciones** de la página **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la página **Agregar aplicación**, especifica la siguiente configuración y selecciona **Revisar + agregar** y, a continuación, selecciona **Agregar**:

    |Configuración|Valor|
    |---|---|
    |Origen de aplicación|**Menú Inicio**|
    |Aplicación|**Word**|
    |Nombre para mostrar|**Microsoft Word**|
    |Descripción|**Microsoft Word**|
    |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la página **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la página **Agregar aplicación**, especifica la siguiente configuración y selecciona **Revisar + agregar** y, a continuación, selecciona **Agregar**:

    |Configuración|Valor|
    |---|---|
    |Origen de aplicación|**Menú Inicio**|
    |Aplicación|**Excel**|
    |Nombre para mostrar|**Microsoft Excel**|
    |Descripción|**Microsoft Excel**|
    |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la página **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la página **Agregar aplicación**, especifica la siguiente configuración y selecciona **Revisar + agregar** y, a continuación, selecciona **Agregar**:

    |Configuración|Valor|
    |---|---|
    |Origen de aplicación|**Menú Inicio**|
    |Aplicación|**PowerPoint**|
    |Nombre para mostrar|**Microsoft PowerPoint**|
    |Descripción|**Microsoft PowerPoint**|
    |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la página **Crear un grupo de aplicaciones**, selecciona **Siguiente: Tareas >**.
1. En la pestaña **Tareas** de la página **Crear un grupo de aplicaciones**, selecciona **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la página **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, selecciona **Grupos**, escribe el nombre completo del grupo **AVD-RemoteApp** que identificaste en la primera tarea de este ejercicio, activa la casilla situada junto al nombre de grupo y haz clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la página **Crear un grupo de aplicaciones**, selecciona **Siguiente: Área de trabajo >**.
1. En la pestaña **Área de trabajo** de la página **Crear un área de trabajo**, especifica la siguiente configuración y selecciona **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la página **Crear grupo de aplicaciones**, selecciona **Crear**.

    > **Nota**: espera a que se cree el grupo de aplicaciones. Debería tardar menos de un minuto. 

    > **Nota**: ahora crearás un grupo de aplicaciones basado en la ruta de acceso del archivo como origen de la aplicación.

1. En el explorador web donde se muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, selecciona **Grupos de aplicaciones**.
1. En la página **Azure Virtual Desktop \| Grupos de aplicaciones**, selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la página **Crear un grupo de aplicaciones**, especifica la siguiente configuración y selecciona **Siguiente: Aplicaciones >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-21e-RG**|
    |Grupo de hosts|**az140-21-hp1**|
    |Tipo de grupo de aplicaciones|**RemoteApp**|
    |Nombre del grupo de aplicaciones|**az140-21-hp1-Utilities-RAG**|

1. En la pestaña **Aplicaciones** de la página **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la página **Agregar aplicación**, en la pestaña **Datos básicos**, especifica la siguiente configuración y selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Origen de aplicación|**Ruta de acceso del archivo**|
    |Ruta de acceso de la aplicación|**C:\Windows\system32\cmd.exe**|
    |Identificador de la aplicación|**Símbolo del sistema**|
    |Nombre para mostrar|**Símbolo del sistema**|
    |Descripción|**Símbolo del sistema de Windows**|
    |Requiere línea de comandos|**No**|

1. En la pestaña **Icono**, especifica la siguiente configuración y selecciona **Revisar + agregar**y, a continuación, selecciona **Agregar**:

    |Configuración|Valor|
    |---|---|
    |Ruta de icono|**C:\Windows\system32\cmd.exe**|
    |Índice de icono|0|

1. De nuevo en la pestaña **Aplicaciones** de la página **Crear un grupo de aplicaciones**, selecciona **Siguiente: Tareas >**.
1. En la pestaña **Tareas** de la página **Crear un grupo de aplicaciones**, selecciona **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la página **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, selecciona **Grupos**, escribe el nombre completo del grupo **AVD-RemoteApp** que identificaste en la primera tarea de este ejercicio, activa la casilla situada junto al nombre de grupo y haz clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la página **Crear un grupo de aplicaciones**, selecciona **Siguiente: Área de trabajo >**.
1. En la pestaña **Área de trabajo** de la página **Crear un área de trabajo**, especifica la siguiente configuración y selecciona **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la página **Crear grupo de aplicaciones**, selecciona **Crear**.

    > **Nota**: espera a que se cree el grupo de aplicaciones. Debería tardar menos de un minuto. 

#### Tarea 4: Creación de un área de trabajo de Azure Virtual Desktop

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, selecciona **Áreas de trabajo**.
1. En la página **Azure Virtual Desktop \| Áreas de trabajo**, selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la página **Crear un área de trabajo**, especifica la siguiente configuración y selecciona **Siguiente: Grupos de aplicaciones >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-21e-RG**|
    |Nombre del área de trabajo|**az140-21-ws1**|
    |Nombre descriptivo|**az140-21-ws1**|
    |Ubicación|Nombre de la región de Azure en la que implementaste los recursos en el primer ejercicio de este laboratorio o región cercana a la misma|

1. En la pestaña **Grupos de aplicaciones** de la página **Crear un área de trabajo**, especifica la siguiente configuración:

    |Configuración|Valor|
    |---|---|
    |Registro de un grupo de aplicaciones|**Sí**|

1. En la pestaña **Área de trabajo** de la página **Crear un área de trabajo**, selecciona **+ Registrar grupos de aplicaciones**.
1. En la página **Agregar grupos de aplicaciones**, selecciona el signo más junto a las entradas **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** y **az140-21-hp1-Utilities-RAG** y haz clic en **Seleccionar**. 
1. De nuevo en la pestaña **Grupos de aplicaciones** de la página **Crear un área de trabajo**, selecciona **Revisar y crear**.
1. En la pestaña **Revisar y crear** de la página **Crear un área de trabajo**, selecciona **Crear**.

#### Tarea 5: Concesión de acceso a grupos de hosts de Azure Virtual Desktop

> **Nota**: al usar hosts de sesión unidos a Microsoft Entra, debes asignar a los usuarios y administradores de Azure Virtual Desktop los roles adecuados de control de acceso basado en roles de Azure (RBAC). En concreto, el rol *Inicio de sesión de usuario de máquina virtual* es necesario para iniciar sesión en los hosts de sesión y el rol *Inicio de sesión de administrador de máquina virtual* es necesario para los privilegios administrativos locales. 

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Grupos de recursos** y, en la página **Grupos de recursos**, selecciona **az140-21e-RG**.
1. En la página **az140-21e-RG**, en el menú de navegación vertical, selecciona **Control de acceso (IAM)**.
1. En la página **az140-21e-RG\|Control de acceso (IAM)**, selecciona **+ Agregar** y, a continuación, en el menú desplegable, selecciona **Agregar asignación de roles**.
1. En la pestaña **Rol** de la página **Agregar asignación de roles**, asegúrate de que está seleccionada la pestaña **Roles de función de trabajo**, en el cuadro de texto de búsqueda, escribe **Inicio de sesión de usuario de máquina virtual**, en la lista de resultados, selecciona **Inicio de sesión de usuario de máquina virtual** y, a continuación, selecciona **Siguiente**.
1. En la pestaña **Miembros** de la página **Agregar asignación de roles**, asegúrate de que la opción **Usuario, grupo o entidad de servicio** está seleccionada, haz clic en **+ Seleccionar miembros**, en el panel **Seleccionar miembros**, busca el grupo **AVD-RemoteApp** que identificaste en la primera tarea de este ejercicio y haz clic en **Seleccionar**.
1. En la pestaña **Miembros** de la página **Agregar asignación de roles**, selecciona **Siguiente**.
1. En la pestaña **Tipo de asignación** de la página **Agregar asignación de roles**, establece el **Tipo de asignación** en **Activo** y, a continuación, selecciona **Revisar y asignar**.
1. En la pestaña **Revisar y asignar** de la página **Agregar asignación de roles**, selecciona **Revisar y asignar**. 
1. De nuevo en la página **az140-21e-RG\|Control de acceso (IAM)**, selecciona **+ Agregar** y, en el menú desplegable, selecciona **Agregar asignación de roles**.
1. En la pestaña **Rol** de la página **Agregar asignación de roles**, asegúrate de que está seleccionada la pestaña **Roles de función de trabajo**; en el cuadro de texto de búsqueda, escribe **Inicio de sesión de administrador de máquinas virtuales**, en la lista de resultados, selecciona **Inicio de sesión de administrador de máquinas virtuales** y, a continuación, selecciona **Siguiente**.
1. En la pestaña **Miembros** de la página **Agregar asignación de roles**, asegúrate de que la opción **Usuario, grupo o entidad de servicio** está seleccionada, haz clic en **+ Seleccionar miembros**, en el panel **Seleccionar miembros**, busca el grupo **AVD-DAG** que identificaste en la primera tarea de este ejercicio y haz clic en **Seleccionar**.
1. De nuevo en la pestaña **Miembros** de la página **Agregar asignación de roles**, establece **Tipo de asignación** en **Activo**, selecciona **Revisar y asignar** y luego vuelve a seleccionar **Revisar y asignar**.. 
