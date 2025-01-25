---
lab:
  title: 'Laboratorio: Administración de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Laboratorio: Administración de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)
# Manual de laboratorio para estudiantes

## Dependencias de laboratorio

- Una suscripción a Azure que usarás en este laboratorio.
- Una cuenta de usuario de Microsoft Entra con el rol Propietario o Colaborador en la suscripción de Azure que vas a usar en este laboratorio y con los permisos suficientes para unir dispositivos al inquilino de Entra asociado con esa suscripción a Azure.
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*

## Tiempo estimado

30 minutos

## Escenario del laboratorio

Tienes un entorno de Azure Virtual Desktop existente. Debes configurar el grupo de hosts con hosts de sesión unidos a Microsoft Entra para admitir una variedad de requisitos funcionales y empresariales. Estos requisitos incluyen:

- implementar hosts de sesión adicionales para dar cabida a un mayor número de usuarios remotos
- Minimizar el coste del entorno de Azure Virtual Desktop mediante la optimización de la configuración del equilibrio de carga del grupo de hosts y el aprovechamiento de la funcionalidad *Iniciar máquina virtual al conectarse*
- Maximizar la disponibilidad de los hosts de sesión durante el horario comercial mediante la implementación de ventanas de mantenimiento
- Habilitar el inicio de sesión único en hosts de sesión unidos a Microsoft Entra
- Maximizar la facilidad de uso y la experiencia del usuario (como la reconexión automática de sesiones desconectadas)

## Objetivos
  
Después de completar este laboratorio, podrás:

- Configurar hosts de sesión de Azure Virtual Desktop unidos a Microsoft Entra para admitir una variedad de requisitos funcionales y empresariales

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Administración de un entorno de Azure Virtual Desktop que contiene hosts de sesión unidos a Microsoft Entra
  
Las tareas principales de este ejercicio son las siguientes:

1. Adición de hosts de sesión a un grupo de hosts de Azure Virtual Desktop
1. Revisión y configuración de las propiedades del grupo de hosts
1. Asignación del rol RBAC necesario a una entidad de servicio de Azure Virtual Desktop
1. Configuración de actualizaciones programadas del agente
1. Configuración de propiedades de RDP del grupo de hosts

#### Tarea 1: Implementación de hosts de sesión de grupos de hosts de Azure Virtual Desktop adicionales

1. De ser necesario, en el equipo de laboratorio, inicia un explorador web, ve a Azure Portal e inicia sesión con las credenciales de una cuenta de usuario con el rol de propietario en la suscripción que usarás en este laboratorio.

    > **Nota**: usa las credenciales de la cuenta `User1-` que aparecen en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.

1. En el navegador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, en la barra de menú vertical, en la sección **Administrar**, selecciona **Grupos de hosts**.
1. En la página **Azure Virtual Desktop \| Grupos de hosts**, en la lista de grupos de host, selecciona **az140-21-hp1**.
1. En la página **az140-21-hp1**, en la barra de menús vertical, en la sección **Administrar**, selecciona **Hosts de sesión** y comprueba que el grupo consta de dos hosts. 
1. En la página **az140-21-hp1 \| Hosts de sesión**, selecciona **+Agregar**.
1. En la pestaña **Datos básicos** de la página **Agregar máquinas virtuales al grupo de hosts**, revisa los ajustes preconfigurados y selecciona **Siguiente: Máquinas virtuales**.
1. En la pestaña **Máquinas virtuales** de la página **Agregar máquinas virtuales a un grupo de hosts**, especifica la siguiente configuración y selecciona **Revisar y crear** (deja los otros valores con su configuración predeterminada):

    > **Nota**: al establecer el valor de **prefijo de nombre**, cambia a la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio e identifica la cadena de caracteres entre *User1-* y el carácter *@*. Usa esta cadena para reemplazar el marcador de posición *aleatorio*.

    |Configuración|Valor|
    |---|---|
    |Grupo de recursos|**az140-21e-RG**|
    |Prefijo de nombre|**sh**-*random*|
    |Ubicación de la máquina virtual|Nombre de la región de Azure en la que implementaste las dos primeras máquinas virtuales del host de sesión|
    |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
    |Tipo de seguridad|**Máquina virtual de inicio seguro**|
    |Imagen|**Windows 11 Enterprise para sesiones múltiples, versión 23H2 + Aplicaciones de Microsoft 365**|
    |Tamaño de la máquina virtual|**Standard DC2s_v3**|
    |Número de máquinas virtuales|**1**|
    |Tipo de disco del sistema operativo|**SSD estándar**|
    |Tamaño de disco del SO|**Tamaño predeterminado (128GB)**|
    |Diagnósticos de arranque|**Habilitar con la cuenta de almacenamiento administrada (recomendado)**|
    |Red virtual|**az140-vnet11e**|
    |Subred|**hp1-Subnet**|
    |Grupo de seguridad de red|**Basic**|
    |Puertos de entrada públicos|**No**|
    |Selecciona el directorio al que quieras unirte|**Microsoft Entra ID**|
    |Inscripción de una máquina virtual con Intune|**No**|
    |Nombre de usuario|**Estudiante**|
    |Contraseña|La misma contraseña que usaste al implementar los hosts de sesión en el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)* 
    |Confirmar contraseña|La misma contraseña que especificaste anteriormente|

    > **Nota**: la contraseña debe tener al menos 12 caracteres de longitud y consta de una combinación de caracteres en minúsculas, mayúsculas, dígitos y caracteres especiales. Para obtener más información, consulta la información sobre [los requisitos de contraseña al crear una VM de Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

    > **Nota**: como es probable que hayas observado, es posible cambiar la imagen y el prefijo de las máquinas virtuales a medida que agregas hosts de sesión al grupo existente. En general, esto no se recomienda a menos que planees reemplazar todas las máquinas virtuales del grupo. 

1. En la pestaña **Revisar y crear** de la página **Agregar máquinas virtuales a un grupo de hosts**, selecciona **Crear**.

    > **Nota**: no esperes a que se complete el proceso de aprovisionamiento, sino que avanza a la siguiente tarea. El proceso de aprovisionamiento puede tardar unos 20 minutos. 

#### Tarea 2: Revisión y configuración de las propiedades del grupo de hosts

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos de hosts** y, en la página **Azure Virtual Desktop \| Grupos de hosts**, selecciona **az140-21-hp1**. 
1. En la página **az140-21-hp1**, selecciona **Propiedades** en la sección **Configuración**.
1. En la página **az140-21-hp1\| Propiedades**, revisa las opciones de configuración disponibles, entre las que se incluyen:

    - **Tipo de grupo de aplicaciones preferido**: esta opción establece el tipo de grupo de aplicaciones preferido para el grupo de hosts en **Escritorio** o **RemoteApp**. Si los usuarios finales tienen aplicaciones de Escritorio y RemoteApp publicadas en el grupo de hosts, solo verán el tipo de aplicación seleccionado en su fuente.
    - **Iniciar máquina virtual al conectarse**: habilitar esta opción permite a los usuarios iniciar máquinas virtuales individuales en el grupo de hosts desde el estado desasignado.
    - **Entorno de validación**: el grupo de hosts de validación está pensado para probar los cambios del servicio antes de implementarlos en producción.
    - **Algoritmo de equilibrio de carga**: esta opción proporciona la elección entre el equilibrio de carga en amplitud y en profundidad. El equilibrio de carga en amplitud distribuye nuevas sesiones de usuario entre todos los hosts de sesión disponibles del grupo de hosts. El equilibrio de carga en profundidad distribuye nuevas sesiones de usuario a un host de sesión disponible con el máximo número de conexiones, pero que no ha alcanzado su umbral del límite de sesiones máximo.

1. En la página **az140-21-hp1\| Propiedades**, en la lista desplegable **Algoritmo de equilibrio de carga**, selecciona **Equilibrio de carga en profundidad**.
1. En el cuadro de texto **Límite máximo de sesión**, escribe **8**.
1. En la página **az140-21-hp1\|Propiedades**, establece **Iniciar máquina virtual al conectarse** a **Sí**.

    > **Nota**: *Iniciar máquina virtual al conectarse* te permite reducir los costes, ya que permite que los usuarios finales activen las máquinas virtuales (MV) que se usan como hosts de sesión solo cuando sean necesarias. En el caso de los grupos de hosts personales, *Iniciar máquina virtual al conectarse* solo activará una máquina virtual de host de sesión existente que ya esté asignada o que pueda asignarse a un usuario. En el caso de los grupos de hosts agrupados, *Iniciar máquina virtual al conectarse* solo activará una máquina virtual de host de sesión cuando ninguna esté activada, mientras que las máquinas adicionales solo se activarán cuando la primera máquina virtual alcance el límite de sesión.

1. En la página **az140-21-hp1\| Propiedades**, selecciona **Guardar**.

    > **Nota**: el uso de *Iniciar máquina virtual al conectarse* requiere la asignación del rol de control de acceso basado en roles (RBAC) de *Colaborador de activación de virtualización de escritorio* a la entidad de servicio de *Azure Virtual Desktop* con la suscripción a Azure como ámbito asignable. 

#### Tarea 3: Asignación del rol RBAC necesario a la entidad de servicio de Azure Virtual Desktop

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, inicia una sesión de PowerShell en Azure Cloud Shell.

    > **Nota**: si se te solicita, en el panel **Introducción**, en la lista desplegable **Suscripción**, selecciona el nombre de la suscripción a Azure que usas en este laboratorio y, a continuación, selecciona **Aplicar**.

1. En la sesión de PowerShell del panel Azure Cloud Shell, ejecuta el siguiente comando para recuperar el valor de la propiedad Id. de la suscripción a Azure que usas en este laboratorio y almacenarlo en una variable `$subId`:

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Ejecuta el siguiente comando para crear una variable de $parameters, que almacena una tabla hash que contiene los valores del nombre de definición del rol RBAC, la aplicación Microsoft Entra que representa la entidad de servicio de **Azure Virtual Desktop** y el ámbito de la suscripción:

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Ejecuta el siguiente comando para crear una asignación de rol RBAC:

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Cierre el panel de Cloud Shell.

#### Tarea 4: Configuración de actualizaciones programadas del agente

> **Nota**: la característica Actualizaciones programadas del agente permite crear hasta dos ventanas de mantenimiento para el agente de Azure Virtual Desktop, la pila en paralelo y el agente de supervisión de Geneva para que la actualización se produzca fuera del horario comercial. 

1. En el explorador que muestra Azure Portal, vuelve a la página del grupo de hosts **az140-21-hp1**.
1. En la página **az140-21-hp1**, en la barra de menús vertical, en la sección **Configuración**, selecciona la entrada **Actualizaciones programadas del agente** y, en la página **az140-21-hp1\|Actualizaciones programadas del agente**, activa la casilla **Actualizaciones programadas del agente**.
1. En la sección **Programación**, active la casilla **Usar zona horaria del host de sesión local**.
1. En la sección **Ventana de mantenimiento**, en la lista desplegable **Día**, selecciona **Sábado** y, en la lista desplegable **Hora**, seleccione **11:00 p. m**.
1. Selecciona **Aplicar**.

#### Tarea 5: Configuración de las propiedades de RDP del grupo de hosts

1. En el explorador web que muestra Azure Portal, en la página **az140-21-hp1**, en la barra de menús vertical, en la sección **Configuración**, selecciona la entrada **Propiedades de RDP**.
1. En la pestaña **Información de conexión** de la página **az140-21-hp1\|Propiedades de RDP**, revisa las opciones de configuración disponibles, entre las que se incluyen:

    - **Inicio de sesión único de Microsoft Entra**: esta opción determina si las conexiones intentarán aprovechar la autenticación de Microsoft Entra para iniciar sesión en hosts de sesión unidos a Microsoft Entra y, de forma eficaz, proporcionar una experiencia de inicio de sesión único. Ten en cuenta que no es necesario que el equipo cliente esté unido a Microsoft Entra. 
    - **Credenciales del proveedor de compatibilidad para seguridad**: esta opción controla el uso de CredSSP para la autenticación. CredSSP proporciona la capacidad de reenviar de forma segura las credenciales de usuario desde el dispositivo cliente al host de sesión de Escritorio remoto. Sin embargo, sus funcionalidades no incluyen la compatibilidad con la autenticación Entra ID.
    - **Shell alternativo**: esta opción permite especificar un archivo ejecutable para iniciar cada vez que se establece una nueva conexión a un host de sesión. Esta configuración solo se aplica a los hosts de sesión que ejecutan Windows Server.
    - **Nombre de proxy de KDC**: esta opción proporciona la capacidad de proxy del tráfico de autenticación Kerberos a los controladores de dominio de Active Directory.

    > **Nota**: teniendo en cuenta que tres de estas opciones no son aplicables en nuestro escenario (lo que implica hosts de sesión unidos a Microsft Entra sin presencia de Servicios de dominio de Active Directory), configurarás solo el primero. Esta opción corresponde a la propiedad de RDP `enablerdsaadauth:i:value`.

1. En la lista desplegable **Inicio de sesión único de Microsoft Entra**, selecciona la opción **Conexiones usará la autenticación de Microsoft Entra para proporcionar el inicio de sesión único** y, a continuación, selecciona **Guardar**.

    > **Importante**: es esencial tener en cuenta que habilitar esta propiedad de RDP específica es solo uno de los varios pasos necesarios para implementar la funcionalidad de inicio de sesión único. Otras acciones aplicables a este escenario incluyen habilitar la autenticación de Microsoft Entra para RDP en el inquilino Entra y configurar grupos de dispositivos, que no se admiten en la versión actual del entorno de laboratorio, por lo que no se incluyen en las instrucciones. Para obtener la lista completa de las acciones necesarias para implementar el inicio de sesión único para Microsoft Entra ID, consulta [Configuración del inicio de sesión único para Azure Virtual Desktop mediante la autenticación de Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on).

1. En la página **az140-21-hp1\|Propiedades de RDP**, selecciona la pestaña **Comportamiento de sesión** y revisa las opciones de configuración disponibles, entre las que se incluyen:

    - **Reconexión**: esta opción determina si el equipo cliente intentará automáticamente volver a conectarse al equipo remoto si se interrumpe la conexión.
    - **Detección automática de ancho de banda**: esta opción determina si se debe usar o no la detección automática de ancho de banda de red.
    - **Detección automática de red**: esta opción permite habilitar la detección automática del tipo de red. Se usa junto con **Detección automática de ancho de banda**. 
    - **Compresión**: esta opción determina si la conexión debe usar la compresión en masa.
    - **Reproducción de vídeo**: esta opción permite el uso del streaming multimedia eficaz RDP para la reproducción de vídeo.

1. En la pestaña **Comportamiento de sesión**, en la lista desplegable **Reconexión**, selecciona **Cliente intenta volver a conectarse automáticamente** y, a continuación, selecciona **Guardar**.
1. En la página **az140-21-hp1\|Propiedades de RDP**, selecciona la pestaña **Redirección de dispositivos** y revisa las opciones de configuración disponibles, incluidas las dos categorías principales:

    - **Audio y vídeo**
    - **Dispositivos y recursos locales**

    > **Nota**: de forma predeterminada, el redireccionamiento se aplica a todas las unidades de disco, incluidas las que se montan después de establecer la conexión inicial.

1. En la página **az140-21-hp1\|Propiedades de RDP**, selecciona la pestaña **Configuración de pantalla** y revisa las opciones de configuración disponibles, incluida la compatibilidad con varias pantallas, el ajuste de tamaño inteligente y los tamaños de escritorio específicos (en píxeles). 
1. En la página **az140-21-hp1\|Propiedades de RDP**, selecciona la pestaña **Avanzado** y revisa los valores de configuración existentes. Ten en cuenta que esta configuración refleja los cambios realizados anteriormente en esta tarea.