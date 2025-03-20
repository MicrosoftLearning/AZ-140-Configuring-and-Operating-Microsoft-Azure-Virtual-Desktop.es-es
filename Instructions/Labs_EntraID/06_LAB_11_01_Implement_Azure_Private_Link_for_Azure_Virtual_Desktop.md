---
lab:
  title: 'Laboratorio: Implementación de Azure Private Link para Azure Virtual Desktop'
  module: 'Module 1.1: Plan, implement, and manage networking for Azure Virtual Desktop'
---

# Laboratorio: Implementación de Azure Private Link para Azure Virtual Desktop
# Manual de laboratorio para estudiantes

## Dependencias de laboratorio

- Una suscripción a Azure que usarás en este laboratorio.
- Una cuenta de usuario de Microsoft Entra con el rol de propietario en la suscripción a Azure que vas a usar en este laboratorio y con los permisos suficientes para unir los dispositivos al inquilino de Microsoft Entra asociado a esa suscripción a Azure.
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*

## Tiempo estimado

40 minutos

## Escenario del laboratorio

Tienes un entorno de Azure Virtual Desktop existente. Debes implementar la conexión al entorno mediante Azure Private Link. 

## Objetivos
  
Después de completar este laboratorio, podrás:

- Implementar Azure Private Link para Azure Virtual Desktop

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Implementación de Azure Private Link para Azure Virtual Desktop
  
Las tareas principales de este ejercicio son las siguientes:

1. Re-registro del proveedor de recursos de Azure Virtual Desktop
1. Creación de una subred de red virtual de Azure
1. Implementación de un punto de conexión privado para las conexiones a un grupo de hosts
1. Implementación de un punto de conexión privado para la descarga de fuentes
1. Implementación de un punto de conexión privado para la detección inicial de fuentes
1. Validación de la funcionalidad del punto de conexión privado
1. Permiso de acceso de red pública a un grupo de hosts y un área de trabajo

> **Nota**: Azure Virtual Desktop tiene tres flujos de trabajo con tres tipos de recursos correspondientes de puntos de conexión privados. Estos flujos de trabajo son:

- **Detección inicial de fuentes**: permite a los clientes RDP detectar todas las áreas de trabajo asignadas a un usuario. Para implementar este flujo de trabajo a través de Private Link, debes crear un único punto de conexión privado al subrecurso global en cualquier área de trabajo que forme parte de la implementación de Azure Virtual Desktop. Sin embargo, independientemente del área de trabajo que elijas, solo puede haber un único punto de conexión privado que proporcione esta funcionalidad por implementación.
- **Descarga de fuentes**: permite a los clientes RDP descargar todos los detalles de conexión para las áreas de trabajo que hospedan los grupos de aplicaciones del usuario actual. Para implementar este flujo de trabajo a través de Private Link, debes crear un punto de conexión privado para el subrecurso de fuente para cada área de trabajo que quieras que esté disponible a través del punto de conexión privado.
- **Conexiones a grupos hosts**: permite a los clientes RDP y hosts de sesión conectarse a un grupo de hosts. Para implementar este flujo de trabajo a través de Private Link, debes crear un punto de conexión privado para el subrecurso de conexión para cada grupo de hosts que pretendes que esté disponible a través del punto de conexión privado.

> **Nota**: puedes implementar estos flujos de trabajo en las siguientes disposiciones:

- Todas las partes de la conexión (descubrimiento inicial de fuentes, descarga de fuentes y conexiones de sesión remota para clientes y hosts de sesión) usan rutas privadas.
- Las conexiones de descarga de fuentes y sesiones remotas para clientes y hosts de sesión usan rutas privadas, pero la detección de fuentes iniciales usa rutas públicas. 
- Solo las conexiones de sesión remota para clientes y hosts de sesión usan rutas privadas, pero la detección inicial de fuentes y la descarga de fuentes usan rutas públicas.
- Tanto los clientes como las máquinas virtuales del host de sesión usan rutas públicas. Private Link no se usa en este escenario.

> **Nota**: en este laboratorio, implementarás la primera disposición.

#### Tarea 1: Re-registro del proveedor de recursos de Azure Virtual Desktop

> **Nota**: para poder usar Private Link con Azure Virtual Desktop, debes volver a registrar el proveedor de recursos **Microsoft.DesktopVirtualization**. 

1. De ser necesario, en el equipo de laboratorio, inicia un explorador web, ve a Azure Portal e inicia sesión con las credenciales de una cuenta de usuario con el rol de propietario en la suscripción que usarás en este laboratorio.

    > **Nota**: usa las credenciales de la cuenta `User1-` que aparecen en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Suscripciones**, en la página **Suscripciones**, selecciona la suscripción a Azure que usas en este laboratorio y, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Proveedores de recursos**.
1. En la pestaña **Proveedores de recursos**, en el cuadro de texto de búsqueda, escribe **Microsoft.DesktopVirtualization**, en la lista de resultados, selecciona el círculo pequeño a la izquierda de la entrada **Microsoft.DesktopVirtualization** y, a continuación, selecciona **Volver a registrar**.

    > **Nota**: espera a que finalice el proceso de re-registro. Esto suele tardar menos de un minuto.

#### Tarea 2: Creación de una subred de red virtual de Azure

> **Nota**: puedes usar la subred existente de una red virtual de Azure para implementar puntos de conexión privados en el escenario de laboratorio, pero es una práctica habitual usar una subred dedicada para este fin.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Redes virtuales** y, en la página **Redes virtuales**, selecciona **az140-vnet11e**.
1. En la página **az140-vnet11e**, en la sección **Configuración** del menú de navegación vertical, selecciona **Subredes**.
1. En la página **az140-vnet11e \| Subredes**, selecciona **+ Subred**.
1. En el panel **Agregar una subred**, especifica la siguiente configuración y selecciona **Agregar** (deja los demás con sus valores predeterminados):

    |Configuración|Valor|
    |---|---|
    |Nombre|**pe-Subnet**|
    |Dirección inicial|**10.20.255.0**|
    |Habilitar subred privada (sin acceso de salida predeterminado)|Deshabilitado|

#### Tarea 3: Implementación de un punto de conexión privado para las conexiones a un grupo de hosts

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos de hosts** y, en la página **Azure Virtual Desktop \| Grupos de hosts**, selecciona **az140-21-hp1**. 
1. En la página **az140-21-hp1**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Redes**.
1. En la página **az140-21-hp1 \| Redes**, selecciona la pestaña **Conexiones de punto de conexión privado** y, después, selecciona **+ Punto de conexión privado**.
1. En la pestaña **Datos básicos** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Recurso >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-11e-RG**|
    |Nombre|**az140-11-pehp1**|
    |Nombre de la interfaz de red|**az140-11-pehp1-nic**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de laboratorio de Azure Virtual Desktop|

1. En la pestaña **Recursos** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Red virtual >**:

    |Configuración|Valor|
    |---|---|
    |Recurso secundario de destino|**connection**|

1. En la pestaña **Red virtual** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: DNS >** (deja los otros valores con su configuración predeterminada):

    |Configuración|Valor|
    |---|---|
    |Red virtual|**az140-vnet11e (az140-11e-RG)**|
    |Subred|**pe-Subnet**|
    |Directiva de red para los puntos de conexión privados|**Deshabilitado**|
    |Configuración de IP privada|**Asignación dinámica de la dirección IP**|

1. En la pestaña **DNS** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Etiquetas >**:

    |Configuración|Valor|
    |---|---|
    |Integración con una zona DNS privada|**Sí**|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-11e-RG**|

    > **Nota**: este paso dará lugar a la creación de una zona DNS privada denominada **privatelink.wvd.microsoft.com**.

1. En la pestaña **Etiquetas** de la página **Crear punto de conexión privado**, selecciona **Siguiente: Revisar y crear**.
1. En la pestaña **Revisar y crear** de la página **Crear punto de conexión privado**, selecciona **Crear**.

    > **Nota**: espera a que la implementación se complete. La implementación puede tardar unos 3 minutos.

    > **Nota**: debes crear un punto de conexión privado para el recurso secundario de conexión para cada grupo de hosts que quieras usar con Private Link.

#### Tarea 4: Implementación de un punto de conexión privado para la descarga de fuentes

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, selecciona **Áreas de trabajo**.
1. En la página **Azure Virtual Desktop \| Áreas de trabajo**, selecciona **az140-21-ws1**.
1. En la página **az140-21-ws1**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Redes**.
1. En la página **az140-21-ws1 \| Redes**, selecciona **Conexiones de punto de conexión privado** y, después, **+ Nuevo punto de conexión privado**.
1. En la pestaña **Datos básicos** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Recurso >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-11e-RG**|
    |Nombre|**az140-11-pefeeddwnld**|
    |Nombre de la interfaz de red|**az140-11-pefeeddwnld-nic**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de laboratorio de Azure Virtual Desktop|

1. En la pestaña **Recursos** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Red virtual >**:

    |Configuración|Valor|
    |---|---|
    |Recurso secundario de destino|**feed**|

1. En la pestaña **Red virtual** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: DNS >** (deja los otros valores con su configuración predeterminada):

    |Configuración|Valor|
    |---|---|
    |Red virtual|**az140-vnet11e (az140-11e-RG)**|
    |Subred|**pe-Subnet**|
    |Directiva de red para los puntos de conexión privados|**Deshabilitado**|
    |Configuración de IP privada|**Asignación dinámica de la dirección IP**|

1. En la pestaña **DNS** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Etiquetas >**:

    |Configuración|Valor|
    |---|---|
    |Integración con una zona DNS privada|**Sí**|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-11e-RG**|

    > **Nota**: este paso aprovechará la zona DNS privada denominada **privatelink.wvd.microsoft.com** que creaste en la tarea anterior.

1. En la pestaña **Etiquetas** de la página **Crear punto de conexión privado**, selecciona **Siguiente: Revisar y crear**.
1. En la pestaña **Revisar y crear** de la página **Crear punto de conexión privado**, selecciona **Crear**.

    > **Nota**: no esperes a que se completen las implementaciones, sino que avanza a la siguiente tarea. La implementación puede tardar un minuto.

    > **Nota**: debes crear un punto de conexión privado para el recurso secundario de fuente para cada área de trabajo que quieras usar con Private Link.

#### Tarea 5: Implementación de un punto de conexión privado para la detección inicial de fuentes

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, selecciona **Áreas de trabajo**.
1. En la página **Azure Virtual Desktop \| Áreas de trabajo**, selecciona **az140-21-ws1**.
1. En la página **az140-21-ws1**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Redes**.
1. En la página **az140-21-ws1 \| Redes**, selecciona **Conexiones de punto de conexión privado** y, después, **+ Nuevo punto de conexión privado**.
1. En la pestaña **Datos básicos** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Recurso >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-11e-RG**|
    |Nombre|**az140-11-pefeeddisc**|
    |Nombre de la interfaz de red|**az140-11-pefeeddisc-nic**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de laboratorio de Azure Virtual Desktop|

1. En la pestaña **Recursos** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Red virtual >**:

    |Configuración|Valor|
    |---|---|
    |Recurso secundario de destino|**global**|

1. En la pestaña **Red virtual** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: DNS >** (deja los otros valores con su configuración predeterminada):

    |Configuración|Valor|
    |---|---|
    |Red virtual|**az140-vnet11e (az140-11e-RG)**|
    |Subred|**pe-Subnet**|
    |Directiva de red para los puntos de conexión privados|**Deshabilitado**|
    |Configuración de IP privada|**Asignación dinámica de la dirección IP**|

1. En la pestaña **DNS** de la página **Crear un punto de conexión privado**, especifica la siguiente configuración y selecciona **Siguiente: Etiquetas >**:

    |Configuración|Valor|
    |---|---|
    |Integración con una zona DNS privada|**Sí**|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|**az140-11e-RG**|

    > **Nota**: este paso aprovechará la zona DNS privada denominada **privatelink.wvd.microsoft.com** que creaste en una de las tareas anteriores.

1. En la pestaña **Etiquetas** de la página **Crear punto de conexión privado**, selecciona **Siguiente: Revisar y crear**.
1. En la pestaña **Revisar y crear** de la página **Crear punto de conexión privado**, selecciona **Crear**.

    > **Nota**: no esperes a que se completen las implementaciones, sino que avanza a la siguiente tarea. La implementación puede tardar un minuto.

    > **Nota**: debes crear un punto de conexión privado para el recurso secundario de fuente para cada área de trabajo que quieras usar con Private Link.

    > **Nota**: para que los cambios de red surtan efecto, debes reiniciar los hosts de sesión en el grupo de hosts de destino.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, ve a la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos de hosts** y, en la página **Azure Virtual Desktop \| Grupos de hosts**, selecciona **az140-21-hp1**.
1. En la página **az140-21-hp1**, en la sección **Administrar** del menú de navegación vertical, selecciona **Hosts de sesión**. 
1. En la lista de hosts de sesión, activa todas las casillas situadas a la izquierda de cada host de sesión y, a continuación, selecciona **Reiniciar** en la barra de herramientas.

    > **Nota**: espera hasta que todos los hosts de sesión estén en estado de **ejecución**. 

#### Tarea 6: Validación de la funcionalidad del punto de conexión privado

> **Nota**: de forma predeterminada, se permite la conectividad con áreas de trabajo de Azure Virtual Desktop y grupos de hosts desde redes públicas. Comenzarás cambiando la configuración predeterminada y aplicando el acceso privado.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, selecciona **Áreas de trabajo**.
1. En la página **Azure Virtual Desktop \| Áreas de trabajo**, selecciona **az140-21-ws1**.
1. En la página **az140-21-ws1**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Redes**.
1. En la página **az140-21-ws1 \| Redes**, en la pestaña **Acceso público**, selecciona la opción **Deshabilitar el acceso público y usar el acceso privado** y, a continuación, selecciona **Guardar**.
1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos de hosts** y, en la página **Azure Virtual Desktop \| Grupos de hosts**, selecciona **az140-21-hp1**. 
1. En la página **az140-21-hp1**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Redes**.
1. En la página **az140-21-hp1 \| Redes**, en la pestaña **Acceso público**, selecciona la opción **Deshabilitar acceso público y usar acceso privado** y, a continuación, selecciona **Guardar**.

    > **Nota**: para validar la funcionalidad del punto de conexión privado, un cliente RDP debe estar conectado a una red que tenga conectividad privada a la red virtual de Azure que contiene la subred que hospeda los puntos de conexión privados que creaste anteriormente en este laboratorio. Para simular este escenario, crearás otra subred en la misma red virtual que se usa para crear puntos de conexión privados e implementarás una VM de Azure que ejecuta Windows 11 en esa subred.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Redes virtuales** y, en la página **Redes virtuales**, selecciona **az140-vnet11e**.
1. En la página **az140-vnet11e**, en la sección **Configuración** del menú de navegación vertical, selecciona **Subredes**.
1. En la página **az140-vnet11e \| Subredes**, selecciona **+ Subred**.
1. En el panel **Agregar una subred**, especifica la siguiente configuración y selecciona **Agregar** (deja los demás con sus valores predeterminados):

    |Configuración|Valor|
    |---|---|
    |Nombre|**client-Subnet**|
    |Dirección inicial|**10.20.2.0**|
    |Habilitar subred privada (sin acceso de salida predeterminado)|Deshabilitado|

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Máquinas virtuales**, en la página **Máquinas virtuales**, selecciona **+ Crear** y, en la lista desplegable, selecciona **Máquina virtual de Azure**.
1. En la pestaña **Datos básicos** de la página **Crear una máquina virtual**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada) y selecciona **Siguiente: Discos >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-111e-RG**.|
    |Nombre de la máquina virtual|**az140-111e-vm0**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de laboratorio de Azure Virtual Desktop|
    |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
    |Tipo de seguridad|**Estándar**|
    |Imagen|**Windows 11 Pro, versión 23H2 - x64 Gen2**|
    |Tamaño|**Standard DC2s_v3**|
    |Nombre de usuario|Cualquier nombre de usuario válido de tu elección|
    |Contraseña|Cualquier contraseña válida de tu elección|
    |Puertos de entrada públicos|**Ninguno**|
    |Licencias|Activa la casilla|

    > **Nota**: la contraseña debe tener al menos 12 caracteres de longitud y consta de una combinación de caracteres en minúsculas, mayúsculas, dígitos y caracteres especiales. Para obtener más información, consulta la información sobre [los requisitos de contraseña al crear una VM de Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. En la pestaña **Discos** de la página **Crear una máquina virtual**, establece el **tipo de disco del sistema operativo** en **HDD estándar (almacenamiento con redundancia local)** y selecciona **Siguiente: Redes >**.
1. En la pestaña **Redes** de la página **Crear una máquina virtual**, especifica las siguientes opciones de configuración (deja los otros valores con su configuración predeterminada):

    |Configuración|Valor|
    |---|---|
    |Red virtual|**az140-vnet11e**|
    |Subred|**client-Subnet**|
    |Dirección IP pública|**(nuevo) az140-111e-vm0-ip**|
    |Grupo de seguridad de red de NIC|**Avanzado**|

1. En la pestaña **Redes** de la página **Crear una máquina virtual**, al lado de la lista desplegable **Configurar grupo de seguridad de red**, selecciona **Crear nuevo**.
1. En la página **Crear grupo de seguridad de red**, elimina la regla entrante creada previamente **1000: default-allow-rdp** y, a continuación, selecciona **+ Agregar una regla entrante**.
1. En el panel **Agregar regla de seguridad entrante**, en la lista desplegable **Origen**, selecciona **Mi dirección IP** para identificar la dirección IP pública que representa la conexión a Internet.
1. En el panel **Agregar una regla de seguridad entrante**, especifica la siguiente opción de configuración y, a continuación, selecciona **Agregar** (deja las demás opciones con sus valores predeterminados):

    |Configuración|Valor|
    |---|---|
    |Origen|**Direcciones IP**|
    |Intervalos de direcciones IP de origen y CIDR|Deja sin cambios (esto debe contener la dirección IP pública)|
    |Intervalos de puertos de origen|*|
    |Destino|**Cualquiera**|
    |Servicio|**RDP**|
    |Acción|**Permitir**|
    |Priority|**300**|
    |Nombre|**AllowCidrBlockRDPInbound**|

1. En la página **Crear grupo de seguridad de red**, selecciona **Aceptar**.
1. De nuevo en la pestaña **Redes** de la página **Crear una máquina virtual**, selecciona **Siguiente: Administración >**:
1. En la pestaña **Administración** de la página **Crear una máquina virtual**, especifica la siguiente configuración (deja los otros valores con su configuración predeterminada) y selecciona **Siguiente: Supervisión >**:

    |Configuración|Valor|
    |---|---|
    |Habilitar el plan básico de forma gratuita|deshabilitado|
    |Opciones de orquestación de revisiones|**Actualizaciones manuales**|

1. En la pestaña **Supervisión** de la página **Crear una máquina virtual**, especifica la siguiente opción de configuración y selecciona **Revisar y crear** (deja todas las demás opciones con su valor predeterminado):

    |Configuración|Valor|
    |---|---|
    |Diagnósticos de arranque|**Deshabilitar**|

1. En la pestaña **Revisar y crear** de la página **Crear una máquina virtual**, selecciona **Crear**.

    > **Nota**: espera a que la implementación se complete. La implementación puede tardar unos 5 minutos.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Máquinas virtuales** y, en la página **Máquinas virtuales**, selecciona **az140-111e-vm0**.
1. En la página **az140-111e-vm0**, selecciona **Conectar**, en el menú desplegable, selecciona **Conectar**
1. En la página **az140-111e-vm0 \| Conectar**, en la sección **Más común**, selecciona **Descargar archivo RDP**.
1. En la ventana emergente **Descargar**, selecciona **Mantener** y, a continuación, selecciona **Abrir archivo**.
1. Cuando se te solicite, selecciona **Conectar** y, a continuación, en el cuadro de diálogo **Seguridad de Windows**, escribe el nombre de usuario y la contraseña que especificaste al implementar la VM de Azure.
1. Cuando se te pida confirmación, vuelve a selecciona **Conectar**.
1. Dentro de la sesión de Escritorio remoto a **az140-111e-vm0**, elige y acepta la configuración de privacidad preferida.
1. En la sesión de Escritorio remoto a **az140-111e-vm0**, inicia Microsoft Edge, ve a la página [Conectar a Azure Virtual Desktop con el cliente de Escritorio remoto para Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), desplázate hacia abajo hasta la sección **Descargar e instalar el cliente de Escritorio remoto (MSI)** y selecciona el vínculo de [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2139369). 
1. Abre Explorador de archivos, ve a la carpeta **Descargas** e inicia la instalación del archivo MSI recién descargado. 
1. Cuando se te solicite, acepta los términos del contrato de licencia y elige la opción **Instalar para todos los usuarios de esta máquina**. Si se te solicita, acepta el menaje del control de cuentas de usuario para continuar con la instalación. 
1. Una vez completada la instalación, asegúrate de que la casilla **Iniciar Escritorio remoto cuando se cierre la instalación** esté activada y haz clic en **Finalizar** para iniciar el cliente de Escritorio remoto de Microsoft.
1. En la sesión de Escritorio remoto a **az140-111e-vm0**, en la ventana cliente de **Escritorio remoto**, selecciona **Suscribirse** y, cuando se te solicite, inicia sesión con las credenciales de la `User2` cuenta de usuario de Entra ID que puedes encontrar en la pestaña **Recursos** del panel derecho de la ventana de la interfaz de laboratorio.

   > **Nota**: selecciona la cuenta de usuario que es miembro del grupo Entra con el prefijo **AVD-RemoteApp**.

1. Asegúrate de que en la página **Escritorio remoto** se muestran cuatro iconos, incluidos el símbolo del sistema, Microsoft Word, Microsoft Excel y Microsoft PowerPoint. 
1. Haz doble clic en el icono del símbolo del sistema. 
1. Cuando se te pida que inicies sesión, en el cuadro de diálogo **Seguridad de Windows**, escribe la contraseña de la misma cuenta de usuario de Microsoft Entra que usaste para conectarte al entorno de Azure Virtual Desktop de destino.
1. Comprueba que aparece una ventana del **símbolo del sistema** poco después. 
1. En el Símbolo del sistema, escribe **logoff** y presiona la tecla **Intro** para cerrar sesión desde la sesión de aplicación remota actual.

   > **Nota**: opcionalmente, puedes considerar la posibilidad de intentar suscribirte a la fuente y conectarte al área de trabajo de Azure Virtual Desktop desde el equipo de laboratorio para validar que se producirá un error en esta conexión. 

    > **Nota**: para minimizar los cargos asociados con la ejecución del entorno de laboratorio, detendrás y desasignarás la VM de Azure recién aprovisionada.

#### Tarea 7: Permiso para acceder a la red pública para un grupo de hosts y un área de trabajo

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, selecciona **Áreas de trabajo**.
1. En la página **Azure Virtual Desktop \| Áreas de trabajo**, selecciona **az140-21-ws1**.
1. En la página **az140-21-ws1**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Redes**.
1. En la página **az140-21-ws1 \| Redes**, en la pestaña **Acceso público**, selecciona la opción **Habilitar acceso público desde todas las redes** y, a continuación, selecciona **Guardar**.
1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Grupos de hosts** y, en la página **Azure Virtual Desktop \| Grupos de hosts**, selecciona **az140-21-hp1**. 
1. En la página **az140-21-hp1**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Redes**.
1. En la página **az140-21-hp1 \| Redes**, en la pestaña **Acceso público**, selecciona la opción **Habilitar acceso público desde todas las redes** y, a continuación, selecciona **Guardar**.
