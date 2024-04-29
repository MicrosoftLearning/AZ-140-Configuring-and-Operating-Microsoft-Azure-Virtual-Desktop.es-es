---
lab:
  title: "Laboratorio: Implementación del escalado automático en grupos de hosts (AD\_DS)"
  module: 'Module 5: Monitor and Maintain an AVD Infrastructure'
---

# Laboratorio: Implementación del escalado automático en grupos de hosts (AD DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Microsoft Entra asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**
- Haber completado el laboratorio **Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (AD DS)**

## Tiempo estimado

60 minutos

## Escenario del laboratorio

Debe configurar el escalado automático de los hosts de sesión de Azure Virtual Desktop en un entorno de Active Directory Domain Services (AD DS).

## Objetivos
  
Después de completar este laboratorio, podrá:

- Configurar el escalado automático de hosts de sesión de Azure Virtual Desktop
- Verificar el escalado automático de hosts de sesión de Azure Virtual Desktop

## Archivos de laboratorio

- None

## Instrucciones

### Ejercicio 1: Configurar el escalado automático de hosts de sesión de Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Preparación para escalar hosts de sesión de Azure Virtual Desktop
2. Configuración de diagnósticos para realizar un seguimiento del escalado automático de Azure Virtual Desktop
3. Creación de un plan de escalado para hosts de sesión de Azure Virtual Desktop

#### Tarea 1: Preparación para escalar hosts de sesión de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En el equipo de laboratorio, en la ventana del explorador web donde se muestra Azure Portal, abra una sesión de **PowerShell** en el panel de **Cloud Shell**.

   >**Nota**: Los grupos de hosts que planea usar con escalabilidad automática deben configurarse con un valor no predeterminado del **parámetro MaxSessionLimit**. Puede establecer este valor en la configuración del grupo de host en Azure Portal o ejecutando los cmdlets **Update-AzWvdHostPool** de Azure PowerShell, como en este ejemplo. También puede establecerlo explícitamente al crear un grupo en Azure Portal o al ejecutar el cmdlet **New-AzWvdHostPool**. cmdlet de Azure PowerShell.

1. Desde la sesión de PowerShell en el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor del parámetro **MaxSessionLimit** del grupo de hosts **az140-21-hp1** en **2**: 

   ```powershell
   Update-AzWvdHostPool -ResourceGroupName 'az140-21-RG' `
   -Name az140-21-hp1 `
   -MaxSessionLimit 2
   ```

   >**Nota**: En este laboratorio, el valor del parámetro **MaxSessionLimit** se establece artificialmente bajo para facilitar el desencadenamiento del comportamiento de escalado automático.

   >**Nota**: Antes de crear su primer plan de escalado, necesitará asignar el rol **Colaborador de encendido y apagado de virtualización de escritorio** de RBAC a Azure Virtual Desktop con su suscripción a Azure como ámbito de destino. 

1. En la ventana del navegador que muestra Azure Portal, cierre el panel de Cloud Shell.
1. En Azure Portal, busque y seleccione **Suscripciones** y, de la lista de suscripciones, seleccione la que contiene los recursos de Azure Virtual Desktop. 
1. En la página de suscripción, seleccione **Control de acceso (IAM)**.
1. En la página **Control de acceso (IAM)**, en la barra de herramientas, seleccione el botón **+ Agregar**, después seleccione **Agregar asignación de rol** en el menú desplegable.
1. En el panel **Agregar asignación de roles** en la pestaña **Rol**, especifique la siguiente configuración y seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Rol de función de trabajo|**Colaborador de encendido y apagado de la virtualización de escritorios**|

1. En el panel **Agregar asignación de funciones**, en la pestaña **Miembros**, haga clic en **+ Seleccionar miembros**, especifique la siguiente configuración y haga clic en **Seleccionar**. 

   |Configuración|Valor|
   |---|---|
   |Seleccionar|**Azure Virtual Desktop** o **Windows Virtual Desktop**|

1. En la hoja **Agregar asignación de roles**, seleccione **Revisar y asignar**

   >**Nota**: El valor depende de cuándo se registró por primera vez el proveedor de recursos **Microsoft.DesktopVirtualization** en su inquilino de Azure.

1. En la pestaña **Revisar y asignar**, seleccione **Revisar y asignar**.

#### Tarea 2: Configuración de diagnósticos para realizar un seguimiento del escalado automático de Azure Virtual Desktop

1. En el equipo de laboratorio, en la ventana del explorador web donde se muestra Azure Portal, abra una sesión de **PowerShell** en el panel de **Cloud Shell**.

   >**Nota**: Usará una cuenta de Azure Storage para almacenar eventos de escalado automático. Puede crearlo directamente desde Azure Portal o usar Azure PowerShell como se muestra en esta tarea.

1. En la sesión de PowerShell del panel Cloud Shell, ejecute los siguientes comandos para crear una cuenta de Azure Storage:

   ```powershell
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName 'az140-11-RG').Location
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   $suffix = Get-Random
   $storageAccountName = "az140st51$suffix"
   New-AzStorageAccount -Location $location -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName Standard_LRS
   ```

   >**Nota**: Espere hasta que se aprovisione la cuenta de almacenamiento.

1. En la ventana del navegador que muestra Azure Portal, cierre el panel de Cloud Shell.
1. En el equipo de laboratorio, en el explorador que muestra Azure Portal, vaya a la página del grupo de hosts**az140-21-hp1**.
1. En la página **az140-21-hp1**, seleccione **Configuración de diagnóstico** y a continuación, seleccione **+ Agregar configuración de diagnóstico**.
1. En la página **Configuración de diagnóstico**, en el cuadro de texto **Nombre de la configuración de diagnóstico**, escriba **az140-51-scaling-plan-diagnostics** y en la sección **Grupos de categorías**, seleccione **Registros de escalado automático para grupos de hosts agrupados**. 
1. En la misma página, en la sección **Detalles del destino**, seleccione **Archivar en una cuenta de almacenamiento** y, en la lista desplegable **Cuenta de almacenamiento**, seleccione el nombre de la cuenta de almacenamiento que empiece por el prefijo **az140st51**.
1. Seleccione **Guardar**.

#### Tarea 3: Creación de un plan de escalado para hosts de sesión de Azure Virtual Desktop

1. En su equipo de laboratorio, en el navegador que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop**. 
1. En la página **Azure Virtual Desktop**, seleccione **Planes de escalado** y después seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** del asistente **Crear un plan de escalado**, especifique la siguiente información y seleccione **Siguiente: Programa >** (deje otros con sus valores predeterminados):

   |Configuración|Value|
   |---|---|
   |Resource group|**az140-51-RG**|
   |Nombre|**az140-51-scaling-plan**|
   |Location|la misma región de Azure en la que implementó los hosts de sesión en los laboratorios anteriores|
   |Nombre descriptivo|**plan de escalado az140-51**|
   |Zona horaria|la zona horaria local|

   >**Nota**: Las etiquetas de exclusión permiten designar un nombre de etiqueta para los hosts de sesión que desea excluir de las operaciones de escalado. Por ejemplo, es posible que quiera etiquetar las máquinas virtuales que están establecidas en modo de purga para que la escalabilidad automática no invalide dicho modo durante el mantenimiento mediante la etiqueta de exclusión "excludeFromScaling". 

1. En la pestaña **Programaciones** del asistente para **Crear un plan de escalado**, seleccione **+ Añadir programación**.
1. En la pestaña **General** del asistente **Añadir programación**, especifique la siguiente información y haga clic en **Siguiente**.

   |Configuración|Valor|
   |---|---|
   |Nombre de programación|**az140-51-schedule**|
   |Repetir|**7 seleccionado** (seleccione todos los días de la semana)|

1. En la pestaña **Aumento** del asistente **Añadir programación**, especifique la siguiente información y haga clic en **Siguiente**.

   |Configuración|Valor|
   |---|---|
   |Hora de inicio (sistema de 24horas)|la hora actual menos 9horas|
   |Algoritmo de equilibrio de carga|**Ancho primero**|
   |Porcentaje mínimo de hosts (%)|**20**|
   |Umbral de capacidad (%)|**60**|

   >**Nota**: La preferencia de equilibrio de carga que seleccione aquí invalidará la que seleccionó para la configuración del grupo de hosts original.

   >**Nota**: El porcentaje mínimo de hosts designa el porcentaje de hosts de sesión en los que desea permanecer siempre activado. Si el porcentaje especificado no es un número entero, se redondea al número entero más cercano. 

   >**Nota**: El umbral de capacidad representa el porcentaje de capacidad del grupo de hosts disponible que desencadenará una acción de escalado que tendrá lugar. Por ejemplo, si dos hosts de sesión del grupo de hosts con un límite máximo de sesión de 20 están activados, la capacidad del grupo de hosts disponible es 40. Si establece el umbral de capacidad en el 75 % y los hosts de sesión tienen más de 30 sesiones de usuario, se activará un tercer host de sesión durante la escalabilidad automática. Esto cambiará la capacidad del grupo de host disponible de 40 a 60.

1. En la pestaña **Horas punta** del asistente **Añadir programación**, especifique la siguiente información y haga clic en **Siguiente**.

   |Configuración|Valor|
   |---|---|
   |Hora de inicio (sistema de 24horas)|la hora actual menos 8 horas|
   |Algoritmo de equilibrio de carga|**Con prioridad a la profundidad**|

   >**Nota**: La hora de inicio designa la hora de finalización de la fase de aumento.

   >**Nota**: El valor de umbral de capacidad de esta fase viene determinado por el valor de umbral de capacidad de aumento.

1. En la pestaña **Descenso** del asistente **Añadir programación**, especifique la siguiente información y haga clic en **Siguiente**.

   |Configuración|Valor|
   |---|---|
   |Hora de inicio (sistema de 24horas)|la hora actual menos 2 horas|
   |Algoritmo de equilibrio de carga|**Con prioridad a la profundidad**|
   |Porcentaje mínimo de hosts (%)|**10**
          |
   |Umbral de capacidad (%)|**90**|
   |Forzar el cierre de sesión de los usuarios|**Sí**|
   |Tiempo de retraso antes de cerrar la sesión de los usuarios y apagar las VM (min)|**30**|

   >**Nota**: Si se habilita la opción **Forzar cierre de sesión de usuarios**, la escalabilidad automática pondrá el host de sesión con el menor número de sesiones de usuario en modo de purga, enviará a todas las sesiones de usuario activas una notificación sobre el cierre inminente y las cerrará por la fuerza una vez transcurrido el tiempo de retraso especificado. Después de que se cierren todas las sesiones de usuario durante la escalabilidad automática, desasigne la máquina virtual. 

   >**Nota**: Si no ha habilitado la desconexión forzada durante el descenso, los hosts de sesión que no tengan sesiones activas o desconectadas serán reasignados.

1. En la pestaña **Horas valle** del asistente **Añadir programación**, especifique la siguiente información y haga clic en **Añadir**.

   |Configuración|Valor|
   |---|---|
   |Hora de inicio (sistema de 24horas)|la hora actual menos 1 hora|
   |Algoritmo de equilibrio de carga|**Con prioridad a la profundidad**|

   >**Nota**: El valor del umbral de capacidad en esta fase viene determinado por el valor del umbral de capacidad de descenso.

1. De nuevo en la pestaña **Programaciones** del asistente de **Creación de un plan de escalado**, seleccione **Siguiente: Asignaciones del grupo de hosts >**:
1. En la página **Asignaciones de grupos de host**, en la lista desplegable **Seleccionar grupo de host**, seleccione **az140-21-hp1**, asegúrese de que la casilla **Habilitar escalabilidad automática** está activada, seleccione **Revisar y crear** y, después, seleccione **Crear**.


### Ejercicio 2: Verificar el escalado automático de hosts de sesión de Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Verificar el escalado automático de hosts de sesión de Azure Virtual Desktop


#### Tarea 1: Verificar el escalado automático de hosts de sesión de Azure Virtual Desktop

1. En el equipo de laboratorio, en la ventana del explorador web donde se muestra Azure Portal, abra una sesión de **PowerShell** en el panel de **Cloud Shell**.
1. Desde la sesión de PowerShell en el panel de Cloud Shell, ejecute el siguiente comando para iniciar la sesión del host de Azure Virtual Desktop de las máquinas virtuales de Azure que usará en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Nota**: Espere hasta que se ejecuten las máquinas virtuales de Azure del host de sesión.

1. En el equipo del laboratorio, en la ventana del explorador web que muestra Azure Portal, navegue hasta la página del grupo de hosts **az140-21-hp1**.
1. En la página **az140-21-hp1**, seleccione **Host de sesión**.
1. Espere hasta que se enumere al menos un host de sesión con el estado **Apagado**.

   > **Nota**: Es posible que tenga que actualizar la página para actualizar el estado de los hosts de sesión.

   > **Nota**: Si todos los hosts de sesión continúan disponibles después de 15 minutos, vuelva a la página **az140-51-scaling-plan** y reduzca el valor de la configuración **Porcentaje mínimo de hosts (%)** **Rampa fuera de servicio**.

   > **Nota**: Una vez que cambie el estado de uno o varios hosts de sesión, los registros de escalado automático deben estar disponibles en la cuenta de Azure Storage. 

1. En Azure Portal, busque y seleccione **Cuentas de almacenamiento** y, en la página **Cuentas de almacenamiento**, seleccione la entrada que representa la cuenta de almacenamiento creada anteriormente en este ejercicio (cuyo nombre comienza con el prefijo **az140st51**).
1. En la página de la cuenta de almacenamiento, seleccione **Contenedores**.
1. En la lista de contenedores, seleccione **insights-logs-autoscaleevaluationpooled**.
1. En la página **insights-logs-autoscaleevaluationpooled**, navegue por la jerarquía de carpetas hasta que llegue a la entrada que representa un blob con formato JSON almacenado en el contenedor.
1. Seleccione la entrada de blob, seleccione el icono de puntos suspensivos situado en el extremo derecho de la página y, en el menú desplegable, seleccione **Descargar**.
1. En el equipo de laboratorio, abra el blob descargado en un editor de texto de su elección y examine su contenido. Debería poder encontrar referencias a eventos de escalado automático y en este caso, podemos buscar "desasignados" para facilitar la identificación.

   >**Nota**: Este es un contenido de blob de ejemplo que incluye referencias a eventos de escalado automático:

   ```json
   host_Ring    "R0"
   Level    4
   ActivityId   "00000000-0000-0000-0000-000000000000"
   time "2023-03-26T19:35:46.0074598Z"
   resourceId   "/SUBSCRIPTIONS/AAAAAAAE-0000-1111-2222-333333333333/RESOURCEGROUPS/AZ140-51-RG/PROVIDERS/MICROSOFT.DESKTOPVIRTUALIZATION/SCALINGPLANS/AZ140-51-SCALING-PLAN"
   operationName    "ScalingEvaluationSummary"
   category "AutoscaleEvaluationPooled"
   resultType   "Succeeded"
   level    "Informational"
   correlationId    "ddd3333d-90c2-478c-ac98-b026d29e24d5"
   properties   
   Message  "Active session hosts are at 0.00% capacity (0 sessions across 3 active session hosts). This is below the minimum capacity threshold of 90%. 2 session hosts can be drained and deallocated."
   HostPoolArmPath  "/subscriptions/aaaaaaaa-0000-1111-2222-333333333333/resourcegroups/az140-21-rg/providers/microsoft.desktopvirtualization/hostpools/az140-21-hp1"
   ScalingEvaluationStartTime   "2023-03-26T19:35:43.3593413Z"
   TotalSessionHostCount    "3"
   UnhealthySessionHostCount    "0"
   ExcludedSessionHostCount "0"
   ActiveSessionHostCount   "3"
   SessionCount "0"
   CurrentSessionOccupancyPercent   "0"
   CurrentActiveSessionHostsPercent "100"
   Config.ScheduleName  "az140-51-schedule"
   Config.SchedulePhase "OffPeak"
   Config.MaxSessionLimitPerSessionHost "2"
   Config.CapacityThresholdPercent  "90"
   Config.MinActiveSessionHostsPercent  "5"
   DesiredToScaleSessionHostCount   "-2"
   EligibleToScaleSessionHostCount  "1"
   ScalingReasonType    "DeallocateVMs_BelowMinSessionThreshold"
   BeganForceLogoffOnSessionHostCount   "0"
   BeganDeallocateVmCount   "1"
   BeganStartVmCount    "0"
   TurnedOffDrainModeCount  "0"
   TurnedOnDrainModeCount   "1"
   ```


### Ejercicio 3: Detener y desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detención y desasignación de máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, anulará la asignación de las máquinas virtuales de Azure usadas en este laboratorio para minimizar las cargas de proceso correspondientes.

#### Tarea 1: Desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel de Cloud Shell, ejecute el comando siguiente para enumerar todas las máquinas virtuales de Azure usadas en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Desde la sesión de PowerShell, en el panel Cloud Shell, ejecute el comando siguiente para detener y desasignar todas las máquinas virtuales de Azure usadas en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.
