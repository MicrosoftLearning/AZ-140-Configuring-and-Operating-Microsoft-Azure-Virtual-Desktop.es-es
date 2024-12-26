---
lab:
  title: 'Laboratorio: Implementación de la supervisión mediante Azure Virtual Desktop Insights'
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Laboratorio: Implementación de la supervisión mediante Azure Virtual Desktop Insights
# Manual de laboratorio para estudiantes

## Dependencias de laboratorio

- Una suscripción a Azure que usarás en este laboratorio.
- Una cuenta de usuario de Microsoft Entra con los roles de Propietario o Colaborador en la suscripción a Azure que usarás en este laboratorio y con los permisos suficientes para unir dispositivos al inquilino de Microsoft Entra asociado a esa suscripción a Azure.
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*

## Tiempo estimado

25 minutos

## Escenario del laboratorio

Tienes un entorno de Azure Virtual Desktop existente. Quieres supervisar el estado y las actividades del entorno.

## Objetivos
  
Después de completar este laboratorio, podrás:

- Implementar la supervisión de un entorno de Azure Virtual Desktop

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Implementación de la supervisión en un entorno de Azure Virtual Desktop
  
Las tareas principales de este ejercicio son las siguientes:

1. Registro de la suscripción a Azure con el proveedor de recursos Microsoft.Insights
1. Creación de un área de trabajo de Azure Log Analytics
1. Configuración del libro de configuración de Virtual Desktop Insights

#### Tarea 1: Registro de la suscripción a Azure con el proveedor de recursos Microsoft.Insights.

> **Nota**: Azure Virtual Desktop Insights se basa en el proveedor de recursos Microsoft.Insights, por lo que primero debes registrarlo en la suscripción a Azure que usas para este laboratorio. En el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID),* has realizado esta tarea con Azure PowerShell. En este laboratorio, lo realizarás mediante Azure Portal (se admite y está disponible cualquiera de los métodos).

1. De ser necesario, en el equipo de laboratorio, inicia un explorador web, ve a Azure Portal e inicia sesión con las credenciales de una cuenta de usuario con el rol de propietario en la suscripción que usarás en este laboratorio.

    > **Nota**: usa las credenciales de la cuenta `User1-` que aparecen en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Suscripciones**, en la página **Suscripciones**, selecciona la suscripción a Azure que usas en este laboratorio y, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Proveedores de recursos**.
1. En la pestaña **Proveedores de recursos**, en el cuadro de texto de búsqueda, escribe **Microsoft.Insights**, en la lista de resultados, selecciona el círculo pequeño a la izquierda de la entrada **Microsoft.Insights** y, a continuación, selecciona **Registrar**.

    > **Nota**: espera a que se complete el proceso de registro. Esto suele tardar un minuto. Usa el botón de la barra de herramientas **Actualizar** para mostrar el valor actualizado del estado de registro.

#### Tarea 2: Creación de un área de trabajo de Azure Log Analytics

> **Nota**: Azure Virtual Desktop Insights es un panel basado en los libros de Azure Monitor que facilita la supervisión de los entornos de Azure Virtual Desktop. 

> **Nota**: solo puedes supervisar las operaciones de escalabilidad automática con Insights con grupos de hosts agrupados. 

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Áreas de trabajo de Log Analytics**, y en la página **Áreas de trabajo de Log Analytics**, selecciona **+ Crear**.
1. En la pestaña **Datos básicos** de la página **Crear un área de trabajo de Log Analytics**, especifica la siguiente configuración y selecciona **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-411e-RG**|
    |Nombre|**az140-laworkspace41e**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|

1. En la página **Revisar y crear**, selecciona **Crear**.

    > **Nota**: espera a que se complete el proceso de aprovisionamiento. Esto suele tardar un minuto.

    > **Nota**: a continuación, debes habilitar la recopilación de datos en el área de trabajo de Log Analytics recién aprovisionada de diagnósticos desde el entorno de Azure Virtual Desktop, los contadores de rendimiento de los hosts de sesión y los registros de eventos de Windows de los hosts de sesión de Azure Virtual Desktop.

#### Tarea 3: Configuración del libro de configuración de Virtual Desktop Insights

> **Nota**: si esta es la primera vez que abres Azure Virtual Desktop Insights, tendrás que configurarlo para tu entorno de Azure Virtual Desktop.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en la página **Azure Virtual Desktop**, en el menú de navegación vertical, en la sección **Supervisión**, selecciona **Libros**.
1. En la lista de libros de **Windows Virtual Desktop**, en la sección **Windows Virtual Desktop**, selecciona el libro **Insights**.
1. En la página **Azure Virtual Desktop \| Libros \| Insights**, revisa los mensajes de advertencia que indican que el área de trabajo y los hosts de sesión no envían datos al área de trabajo y, a continuación, selecciona el vínculo **Libro de configuración** para reparar el problema.
1. En la página **CheckAMAConfiguration**, en la pestaña **Configuración de diagnósticos de recursos**, en la lista desplegable **Área de trabajo de Log Analytics**, selecciona **az140-laworkspace41e**.
1. En la página **CheckAMAConfiguration**, en la pestaña **Configuración de diagnósticos de recursos**, en la sección **Grupo de hosts az140-21-hp1**, ten en cuenta el mensaje de advertencia que indica que no se encontró ninguna configuración de diagnósticos para el grupo de hosts seleccionado y, a continuación, selecciona **Configurar grupo de hosts**.
1. En el panel **Implementar plantilla**, selecciona **Implementar**.

    > **Nota**: esto habilita eficazmente las siguientes tablas de diagnósticos en el área de trabajo de Log Analytics de destino:
    - Actividades de administración
    - Fuente
    - Conexiones
    - Errors
    - Puntos de control
    - HostRegistration
    - AgentHealthStatus

    > **Nota**: espera a que la implementación se complete. Esto suele tardar menos de un minuto.

1. En la página **CheckAMAConfiguration**, en la pestaña **Configuración de diagnósticos de recursos**, selecciona el icono **Actualizar** (una flecha circular) de la barra de herramientas.
1. Revisa la sección **Grupo de hosts Az140-21-hp1** y comprueba que la configuración de diagnósticos está habilitada para **allLogs**.
1. Mientras te encuentras en la pestaña **Configuración de diagnósticos de recursos**, desplázate hacia abajo hasta la sección **Área de trabajo az140-21-ws1** y, a continuación, selecciona **Configurar área de trabajo**.
1. En el panel **Implementar plantilla**, selecciona **Implementar**.

    > **Nota**: esto configura eficazmente el área de trabajo para **allLogs**.

    > **Nota**: espera a que la implementación se complete. Esto suele tardar menos de un minuto.

1. En la página **CheckAMAConfiguration**, en la pestaña **Configuración de diagnósticos de recursos**, selecciona el icono **Actualizar** (una flecha circular) de la barra de herramientas.
1. Revise la sección **Área de trabajo az140-21-ws1** y comprueba que la configuración de diagnósticos está habilitada para **allLogs** y que no quedan mensajes de advertencia restantes.
1. Ve a la parte superior de la página **CheckAMAConfiguration** y cambia a la pestaña **Seleccionar configuración de datos de host**.
1. En la pestaña **Seleccionar configuración de datos de host**, en la sección **Crear DCR**, en la lista desplegable **Destino del área de trabajo**, selecciona **az140-laworkspace41e** y, a continuación, selecciona **Crear regla de recopilación de datos**.
1. En el panel **Implementar plantilla**, selecciona **Implementar**.

    > **Nota**: espera a que la implementación se complete. Esto suele tardar menos de un minuto.

1. En la página **CheckAMAConfiguration**, en la pestaña **Seleccionar configuración de datos de host**, selecciona el icono **Actualizar** (una flecha circular) de la barra de herramientas.

    > **Nota**: antes de continuar, asegúrate de que el DCR recién creado aparece en la subsección **DCR disponibles** de la sección **Crear DCR**. Si no es así, espera otro minuto y actualiza la página de nuevo.

1. En la pestaña **Seleccionar configuración de datos de host**, en la lista desplegable **DCR seleccionado**, selecciona la entrada a partir del prefijo **microsoft-avdi-**.
1. En la pestaña **Seleccionar configuración de datos de host**, en la sección **Asociaciones de DCR**, selecciona **Implementar asociación**.
1. En el panel **Implementar plantilla**, selecciona **Implementar**.

    > **Nota**: esto asocia eficazmente el DCR recién creado a los hosts de sesión en el grupo de hosts **az140-21-hp1**.

    > **Nota**: espera a que la implementación se complete. Esto suele tardar menos de un minuto.

1. En la página **CheckAMAConfiguration**, en la pestaña **Seleccionar configuración de datos de host**, selecciona el icono **Actualizar** (una flecha circular) de la barra de herramientas.
1. En la pestaña **Seleccionar configuración de datos de host**, en la sección **Extensiones de Azure Monitor que le faltan a los hosts de sesión**, selecciona **Agregar extensión**.
1. En el panel **Implementar plantilla**, selecciona **Implementar**.

    > **Nota**: esto instala eficazmente la extensión de Azure Monitor en los hosts de sesión en el grupo de hosts **az140-21-hp1**.

    > **Nota**: espera a que la implementación se complete. Esto puede tardar 1 minuto.

1. En la página **CheckAMAConfiguration**, en la pestaña **Seleccionar configuración de datos de host**, selecciona el icono **Actualizar** (una flecha circular) de la barra de herramientas.
1. Comprueba que no se muestran mensajes de error o advertencia. 
1. Ve a la parte superior de la página **CheckAMAConfiguration**, selecciona la pestaña **Datos generados** y, a continuación, selecciona el icono **Actualizar** (una flecha circular) de la barra de herramientas.
1. Revisa las secciones que muestran gráficos que representan los datos recopilados, incluidos los **datos facturados en las últimas 24 horas**, **contadores de rendimiento** y **eventos**.

    > **Nota**: usa la sección **Datos facturados en los últimos 24 horas** para supervisar la ingesta de datos. Eres responsable de los cargos de Log Analytics por el almacenamiento y la ingesta de datos.

1. En el explorador web que muestra Azure Portal, vuelve a la página de **Azure Virtual Desktop** y, en la sección **Supervisión** del menú de navegación vertical, selecciona **Insights**.
1. En la página **Azure Virtual Desktop \| Insights**, revisa el contenido de la pestaña **Información general**, incluida la sección **Capacidad**, **Diagnósticos de conexión: % de los usuarios que pueden conectarse**, **Rendimiento de la conexión: Tiempo para conectar (nuevas sesiones)** y telemetría de **uso**. 
1. A continuación, revisa todas las pestañas restantes de la página **Azure Virtual Desktop \| Insights**, incluida la **confiabilidad de las conexiones**, los **diagnósticos de conexión**, el **rendimiento de la conexión**, los **usuarios**, el **uso**, los **clientes** y las **alertas**.

    > **Nota**: considera la posibilidad de volver a consultar estas pestañas de la página Insights una vez completados los laboratorios posteriores para revisar los gráficos que representan la telemetría recopilada.
