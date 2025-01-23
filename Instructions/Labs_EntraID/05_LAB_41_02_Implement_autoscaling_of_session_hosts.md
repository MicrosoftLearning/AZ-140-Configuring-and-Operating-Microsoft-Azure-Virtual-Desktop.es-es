---
lab:
  title: 'Laboratorio: Implementación de la escalabilidad automática de hosts de sesión'
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Laboratorio: implementación y supervisión de la escalabilidad automática de hosts de sesión
# Manual de laboratorio para estudiantes

## Dependencias de laboratorio

- Una suscripción a Azure que usarás en este laboratorio.
- Una cuenta de usuario de Microsoft Entra con el rol Propietario o Colaborador en la suscripción de Azure que vas a usar en este laboratorio y con los permisos suficientes para unir dispositivos al inquilino de Entra asociado con esa suscripción a Azure.
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*
- Haber completado el laboratorio *Conexión a hosts de sesión (Entra ID)*

## Tiempo estimado

45 minutos

## Escenario del laboratorio

Tienes un entorno de Azure Virtual Desktop que cambia de uso de forma periódica. Quieres minimizar el coste aprovechando la funcionalidad de los planes de escalabilidad automática.

## Objetivos
  
Después de completar este laboratorio, podrás:

- Implementar y evaluar la escalabilidad automática de Azure Virtual Desktop

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Implementación de planes de escalabilidad automática de Azure Virtual Desktop
  
Las tareas principales de este ejercicio son las siguientes:

1. Asignación del rol RBAC necesario a una entidad de servicio de Azure Virtual Desktop
1. Detención y desasignación de todos los hosts de sesión
1. Ajuste de la configuración del grupo de hosts
1. Creación de un plan de escalado
1. Evaluación de la funcionalidad de escalado automático
1. Deshabilitación del escalado automático del grupo de hosts

#### Tarea 1: Asignación del rol RBAC necesario a la entidad de servicio de Azure Virtual Desktop

> **Nota**: para que los planes de escalabilidad automática funcionen, debes conceder a la entidad de servicio de Azure Virtual Desktop los permisos para administrar el estado de energía de las máquinas virtuales del host de sesión. Estos permisos se pueden conceder mediante el rol RBAC de **Colaborador de encendido y apagado de la virtualización de escritorios**. Es importante tener en cuenta que la asignación de roles debe realizarse en el ámbito de la suscripción. La asignación de este rol en cualquier nivel inferior a la suscripción, como el grupo de recursos, el grupo de hosts o la máquina virtual, impedirá que la escalabilidad automática funcione correctamente. 

> **Nota**: este es diferente del rol **Colaborador de encendido y apagado de la virtualización de escritorios** que se usa en el laboratorio *Administración de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID),* que era necesario para admitir la funcionalidad *Iniciar máquina virtual al conectar*.

1. De ser necesario, en el equipo de laboratorio, inicia un explorador web, ve a Azure Portal e inicia sesión con las credenciales de una cuenta de usuario con el rol de propietario en la suscripción que usarás en este laboratorio.

    > **Nota**: usa las credenciales de la cuenta `User1-` que aparecen en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, inicia una sesión de PowerShell en Azure Cloud Shell.

    > **Nota**: si se te solicita, en el panel **Introducción**, en la lista desplegable **Suscripción**, selecciona el nombre de la suscripción a Azure que usas en este laboratorio y, a continuación, selecciona **Aplicar**.

1. En la sesión de PowerShell del panel Azure Cloud Shell, ejecuta el siguiente comando para recuperar el valor de la propiedad Id. de la suscripción a Azure que usas en este laboratorio y almacenarlo en una variable `$subId`:

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Ejecuta el siguiente comando para crear una variable de $parameters, que almacena una tabla hash que contiene los valores del nombre de definición del rol RBAC, la aplicación Microsoft Entra que representa la entidad de servicio de **Azure Virtual Desktop** y el ámbito de la suscripción:

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Off Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Ejecuta el siguiente comando para crear una asignación de rol RBAC:

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Cierra el panel de Cloud Shell.

#### Tarea 2: Detención y desasignación de todos los hosts de sesión

> **Nota**: para evaluar la funcionalidad de escalado automático, detendrás y desasignarás todos los hosts de sesión en el entorno de Azure Virtual Desktop. 

1. En el equipo de laboratorio, en el navegador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en página **Azure Virtual Desktop**, en la barra de menú vertical, en la sección **Administrar**, selecciona **Grupos de hosts**.
1. En la página **Azure Virtual Desktop \| Grupos de hosts**, en la lista de grupos de host, selecciona **az140-21-hp1**.
1. En la página **az140-21-hp1**, en la barra de menús vertical, en la sección **Administrar**, selecciona **Hosts de sesión**.
1. En la página **az140-21-hp1 \| Hosts de sesión**, activa la casilla situada junto a cada nombre de host de sesión y, a continuación, selecciona **Detener**.
1. Cuando se te pida que confirmes, en la ventana emergente **Detener hosts de sesión**, selecciona **Detener**.

    > **Nota**: es posible que tengas que seleccionar el icono de puntos suspensivos (`...`) de la barra de herramientas para mostrar el botón **Detener**.

    > **Nota**: no esperes hasta que los hosts de sesión se detengan y desasignen, sino que continúa con la siguiente tarea. La detención y desasignación de hosts de sesión puede tardar unos 2 minutos.

#### Tarea 3: Ajuste de la configuración del grupo de hosts

> **Nota**: al usar la escalabilidad automática para grupos de hosts agrupados, debes tener el parámetro MaxSessionLimit configurado para ese grupo de hosts. En este laboratorio, lo establecerás artificialmente bajo para facilitar la ilustración de la funcionalidad de escalado automático.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, en la página **az140-21-hp1**, en la sección **Configuración**, selecciona **Propiedades**.
1. En la página **az140-21-hp1 \|Propiedades**, en el cuadro de texto **Límite máximo de sesión**, escribe **1**.
1. En la página **az140-21-hp1\| Propiedades**, selecciona **Guardar**.

#### Tarea 4: Creación de un plan de escalado

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Planes de escalado**.
1. En la página **Azure Virtual Desktop, seleccione \| Planes de escalado**. selecciona **+ Crear**.
1. En la pestaña **Datos básicos** de la página **Crear un plan de escalado**, especifica la siguiente configuración y selecciona **Siguiente: Programaciones**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **az140-412e-RG**.|
    |Nombre del plan de escalado|**az140-scalingplan412e**|
    |Región|Nombre de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Nombre descriptivo|**az140-scalingplan412e**|
    |Zona horaria|La zona horaria local de la región de Azure donde implementaste el entorno de Azure Virtual Desktop|
    |Tipo de grupo de hosts|**Agrupado**|
    |Método de escalado|**Escalabilidad automática de administración de energía**|

    > **Nota**: deja la propiedad **etiqueta de exclusión** no establecida. En general, puedes usar esta característica para excluir máquinas virtuales de Azure con etiquetas de establecimiento arbitrario de la escalabilidad automática.

1. En la pestaña **Programaciones**, selecciona **+ Agregar programación**.

    > **Nota**: las programaciones te permiten definir horas de aumento, horas punta, horas de disminución y horas de poca actividad durante los días de la semana y especificar desencadenadores de escalado automático. El plan de escalado debe incluir una programación asociada para al menos un día de la semana. 

1. En la pestaña **General** del panel **Agregar una programación**, ajusta la configuración predeterminada para que coincida con la siguiente configuración y, a continuación, selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Zona horaria|La zona horaria local del entorno de Azure Virtual Desktop (en función de la región que seleccionaste anteriormente en esta tarea)|
    |Nombre de programación|**week_schedule**|
    |Repetir|**7 seleccionado** (selecciona todos los días de la semana)|

    > **Nota**: la programación abarca eficazmente todos los días de la semana, lo que facilitará la evaluación del resultado del escalado automático.

1. En la pestaña **Aumento** del panel **Agregar una programación**, ajusta la configuración predeterminada para que coincida con la siguiente configuración y, a continuación, selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Hora de inicio (sistema de 12 horas)|La hora actual menos 1 hora|
    |Algoritmo de equilibrio de carga|**Equilibrio de carga en amplitud**|
    |Porcentaje mínimo de hosts (%)|**30**|
    |Umbral de capacidad (%)|**60**|

    > **Nota**: para los grupos de hosts agrupados, la escalabilidad automática omite los algoritmos de equilibrio de carga existentes en la configuración del grupo de hosts y, en su lugar, aplica el equilibrio de carga en función de la configuración de la programación.

    > **Nota**: la opción **Porcentaje mínimo de hosts** designa el porcentaje mínimo de máquinas virtuales del host de sesión que se iniciarán para las horas de aumento y punta. Por ejemplo, si el **porcentaje mínimo de hosts** se especifica como 30 % y el número total de hosts de sesión del grupo de hosts es 3, la escalabilidad automática garantizará que haya un mínimo de 1 host de sesión disponible para aceptar conexiones de usuario.

    > **Nota**: la escalabilidad automática se redondea hasta el número entero más cercano.

    > **Nota**: la opción **Umbral de capacidad** es el porcentaje de capacidad del grupo de hosts usado que se considerará para evaluar si se deben activar o desactivar las máquinas virtuales durante las horas de aumento y punta. Por ejemplo, si el umbral de capacidad se especifica como 60 % y la capacidad del grupo de hosts es de 1 sesión, la escalabilidad automática activará los hosts de sesión adicionales una vez que el grupo de hosts supere una carga del 60 % (en este caso es del 100 %).

1. En la pestaña **Horas punta** del panel **Agregar una programación**, ajusta la configuración predeterminada para que coincida con la siguiente configuración y, a continuación, selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Hora de inicio (sistema de 12 horas)|La hora actual más 1 hora|
    |Algoritmo de equilibrio de carga|**Equilibrio de carga en profundidad**|
    |Umbral de capacidad (%)|**60**|

    > **Nota**: la opción **Umbral de capacidad (%)** se comparte entre las opciones **Horas de aumento** y **Horas punta**.

1. En la pestaña **Descenso** del panel **Agregar una programación**, ajusta la configuración predeterminada para que coincida con la siguiente configuración y, a continuación, selecciona **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Hora de inicio (sistema de 12 horas)|La hora actual más 2 horas|
    |Algoritmo de equilibrio de carga|**Equilibrio de carga en profundidad**|
    |Porcentaje mínimo de hosts activos (%)|**10**
          |
    |Umbral de capacidad (%)|**80**|
    |Forzar el cierre de sesión de los usuarios|**No**|
    |Detén las máquinas virtuales cuando|**Las máquinas virtuales no tienen sesiones activas o desconectadas**|

    > **Nota**: la opción **Porcentaje mínimo de hosts activos (%)** designa el porcentaje mínimo de máquinas virtuales del host de sesión a las que te gustaría llegar para las horas de aumento y de poca actividad. Por ejemplo, si el **porcentaje mínimo de hosts activos (%)** está establecido en 10 % y el número total de hosts de sesión del grupo de hosts es 3, la escalabilidad automática garantizará que haya un mínimo de 1 host de sesión disponible para tomar conexiones de usuario.

    > **Nota**: la opción **Umbral de capacidad (%)** designa el porcentaje de capacidad del grupo de hosts usado que se considerará para evaluar si se desactivan las máquinas virtuales durante las horas de disminución y de poca actividad Por ejemplo, con 1 conexión de usuario y 3 hosts en ejecución, si se especifica un umbral de capacidad como 80 %, la escalabilidad automática desactivaría 1 host (lo que da lugar al 50 % de la capacidad del grupo de hosts usado).

    > **Nota**: en general, la escalabilidad automática detendrá y desasignará los hosts de sesión según las reglas siguientes:

    - La capacidad utilizada del grupo de hosts está por debajo del umbral de capacidad.
    - La desactivación de los hosts de sesión no provocará que se supere el umbral de capacidad.
    - La escalabilidad automática solo desactiva los hosts de sesión sin sesiones de usuario, a menos que el plan de escalado esté en fase de descenso y hayas habilitado la configuración para forzar el cierre de sesión. 
    - La escalabilidad automática agrupada no desactivará los hosts de sesión en la fase de descenso para evitar una experiencia de usuario incorrecta.

1. En la pestaña **Horas de poca actividad** del panel **Agregar una programación**, ajusta la configuración predeterminada para que coincida con la siguiente configuración y, a continuación, selecciona **Agregar**:

    |Configuración|Valor|
    |---|---|
    |Hora de inicio (sistema de 12 horas)|La hora actual más 3 horas|
    |Algoritmo de equilibrio de carga|**Equilibrio de carga en profundidad**|
    |Umbral de capacidad (%)|**80**|

    > **Nota**: la configuración de **Umbral de capacidad** se comparte entre la configuración de **Horas de aumento** y **horas con poca actividad**.

1. De nuevo en la pestaña **Programaciones** de la página **Creación de un plan de escalado**, selecciona **Siguiente: Asignaciones del grupo de hosts**:
1. En la pestaña **Asignaciones del grupo de host**, en la lista desplegable **Seleccionar grupo de host**, selecciona **az140-21-hp1**, asegúrate de que la casilla **Habilitar escalabilidad automática** está activada, selecciona **Revisar y crear**.
1. En la página **Revisar y crear**, selecciona **Crear**.

    > **Nota**: espera a que se complete la configuración de escalabilidad automática. Por lo general, esto tarda solo unos segundos.

#### Tarea 5: Evaluación de la funcionalidad de escalabilidad automática

> **Nota**: para empezar, evaluarás la configuración de **Aumento**.

1. En el equipo de laboratorio, en el navegador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en página **Azure Virtual Desktop**, en la barra de menú vertical, en la sección **Administrar**, selecciona **Grupos de hosts**.
1. En la página **Azure Virtual Desktop \| Grupos de hosts**, en la lista de grupos de host, selecciona **az140-21-hp1**.
1. En la página **az140-21-hp1**, en la barra de menús vertical, en la sección **Administrar**, selecciona **Hosts de sesión**.
1. En la página **az140-21-hp1 \| Hosts de sesión**, revisa los valores de la configuración de **Estado de energía** de los hosts de sesión y comprueba que uno de ellos aparece como **En ejecución**.

    > **Nota**: es posible que tengas que esperar un par de minutos antes de que los hosts de la primera sesión alcancen el estado **En ejecución**.

    > **Nota**: esto es previsible, ya que según la configuración de **Aumento** del plan de escalado recién creado, al menos un host de sesión debe estar siempre en línea. En este momento, la capacidad del grupo de hosts es 1 (ya que solo hay 1 host en ejecución), pero la capacidad del grupo de hosts usado es del 0 %, ya que no hay conexiones de usuario.

    > **Nota**: a continuación, evaluarás la configuración de Umbral de capacidad de **horas de aumento** y **horas punta** iniciando una sesión de usuario única. Podemos evaluar esto incluso fuera del período de **horas punta**, ya que las dos fases comparten el mismo umbral de capacidad.

1. En el equipo de laboratorio, inicia el cliente de Escritorio remoto de Microsoft.
1. En el equipo de laboratorio, en la ventana cliente de **Escritorio remoto**, selecciona **Suscribirse** y, cuando se te solicite, inicia sesión con las credenciales de la `User2` cuenta de usuario de Entra ID que puedes encontrar en la pestaña **Recursos** del panel derecho de la ventana de la interfaz del laboratorio.
1. Asegúrate de que la página **Escritorio remoto** muestra cuatro iconos, incluidos Microsoft Word, Microsoft Excel, Microsoft PowerPoint y el símbolo del sistema. 
1. Haz doble clic en el icono del símbolo del sistema. 
1. Cuando se te pida que inicies sesión, en el cuadro de diálogo **Seguridad de Windows**, escribe la contraseña de la cuenta de usuario de Microsoft Entra que usaste para conectarte al entorno de Azure Virtual Desktop de destino.
1. Comprueba que aparece una ventana del **símbolo del sistema** poco después. 

    > **Nota**: en este punto, la capacidad utilizada del grupo de hosts es del 100 %, que es mayor que el umbral de capacidad (60 %). Esto debería dar lugar a que la escalabilidad automática active otro host, lo que llevará la capacidad del grupo de hosts usado al 50 %. Dado que se encuentra por debajo del umbral de capacidad, el tercer host permanecerá detenido o desasignado. Comprobarás esto a continuación.

1. En el equipo de laboratorio, cambia al explorador web en la que se muestra Azure Portal. 
1. En la página **az140-21-hp1 \| Hosts de sesión**, selecciona **Actualizar**, revisa los valores de la configuración de **Estado de energía** de los hosts de sesión y comprueba que ahora dos de ellos aparecen como **En ejecución**.

    > **Nota**: a continuación, evaluarás la configuración del umbral de capacidad de **aumento** ajustando su período de tiempo. 

1. En la página **az140-21-hp1 \| Hosts de sesión**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Planes de escalado** y, a continuación, en la página **Planes de escalado**, selecciona **az140-scalingplan412e**.
1. En la página **az140-scalingplan412e**, en el menú de navegación vertical, en la sección **Configuración**, selecciona **Programaciones** y, a continuación, selecciona **week_schedule**.
1. En el panel **week_schedule**, ve a la pestaña **Descenso** y ajusta el valor de la configuración **Hora de inicio (sistema de 12 horas)** en cualquier momento entre la **Hora de inicio (sistema de 12 horas)** de la fase **Horas punta** y la hora actual.

    > **Nota**: es posible que tengas que ajustar el valor de la **Hora de inicio (sistema de 12 horas)** de la fase de **Horas punta**.

1. En el panel **week_schedule**, ve a la pestaña **Horas de poca actividad** y selecciona **Guardar**.
1. Cambia a la ventana del **símbolo del sistema** que representa la única sesión RDP al grupo de hosts y, en el símbolo del sistema, escribe lo siguiente y presiona la tecla **Intro**:

    ```cmd
    logoff
    ```

1. En el equipo de laboratorio, en el navegador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en página **Azure Virtual Desktop**, en la barra de menú vertical, en la sección **Administrar**, selecciona **Grupos de hosts**.
1. En la página **Azure Virtual Desktop \| Grupos de hosts**, en la lista de grupos de host, selecciona **az140-21-hp1**.
1. En la página **az140-21-hp1**, en la barra de menús vertical, en la sección **Administrar**, selecciona **Hosts de sesión**.
1. En la página **az140-21-hp1 \| Hosts de sesión**, revisa los valores de la configuración **Estado de energía** de los hosts de sesión y comprueba que ahora solo se muestra uno de ellos como **En ejecución**.

    > **Nota**: el host de sesión puede tardar entre 1 y 2 minutos en apagarse.

#### Tarea 6: Deshabilitación de la escalabilidad automática del grupo de hosts

> **Nota**: para asegurarte de que la configuración de escalabilidad automática no afectará a otros laboratorios, quitarás la asignación del grupo de hosts del plan de escalado que implementaste en este laboratorio.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop**, en la página **Azure Virtual Desktop**, en la sección **Administrar** del menú de navegación vertical, selecciona **Planes de escalado** y, a continuación, en la página **Planes de escalado**, selecciona **az140-scalingplan412e**.
1. En la página **az140-scalingplan412e**, en la sección **Administrar**, selecciona **Tareas del grupo de hosts**.
1. En la página **az140-scalingplan412e \| Tareas de grupo de hosts**, selecciona **az140-21-hp1** y, después, selecciona **Desasignar** y, cuando se te pida que confirmes, en el cuadro de diálogo **Desasignar grupo de hosts**, selecciona **Desasignar**.