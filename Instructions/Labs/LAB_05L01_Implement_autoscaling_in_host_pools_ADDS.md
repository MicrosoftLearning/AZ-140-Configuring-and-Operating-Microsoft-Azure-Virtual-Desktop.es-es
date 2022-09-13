---
lab:
  title: "Laboratorio: Implementación del escalado automático en grupos de hosts (AD\_DS)"
  module: 'Module 5: Monitor and Maintain a WVD Infrastructure'
---

# <a name="lab---implement-autoscaling-in-host-pools-ad-ds"></a>Laboratorio: Implementación del escalado automático en grupos de hosts (AD DS)
# <a name="student-lab-manual"></a>Manual de laboratorio para alumnos

## <a name="lab-dependencies"></a>Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Azure AD con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Azure AD asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**
- Haber completado el laboratorio **Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (AD DS)**

## <a name="estimated-time"></a>Tiempo estimado

60 minutos

## <a name="lab-scenario"></a>Escenario del laboratorio

Debe configurar el escalado automático de los hosts de sesión de Azure Virtual Desktop en un entorno de Active Directory Domain Services (AD DS).

## <a name="objectives"></a>Objetivos
  
Después de completar este laboratorio, podrá:

- Configurar el escalado automático de hosts de sesión de Azure Virtual Desktop
- Verificar el escalado automático de hosts de sesión de Azure Virtual Desktop

## <a name="lab-files"></a>Archivos de laboratorio

- None

## <a name="instructions"></a>Instructions

### <a name="exercise-1-configure-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Ejercicio 1: Configurar el escalado automático de hosts de sesión de Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Preparar el escalado automático de hosts de sesión de Azure Virtual Desktop
1. Crear y configurar una cuenta de Azure Automation
1. Crear una aplicación de Azure Logic

#### <a name="task-1-prepare-for-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Tarea 1: Preparar el escalado automático de hosts de sesión de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En el equipo de laboratorio, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para iniciar las máquinas virtuales de Azure del host de sesión de Azure Virtual Desktop que vamos a usar en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en iniciarse. 

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>Tarea 2: Crear y configurar una cuenta de Azure Automation

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, haga clic en **az140-dc-vm11**.
1. En la hoja **az140-dc-vm11**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-dc-vm11 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Value|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para iniciar sesión en la suscripción de Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Cuando se le solicite, inicie sesión con las credenciales de Azure AD de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para descargar el script de PowerShell que utilizará para crear la cuenta de Azure Automation que forma parte de la solución de escalado automático:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   New-Item -ItemType Directory -Path $labFilesfolder -Force
   Set-Location -Path $labFilesfolder
   $uri = 'https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1'
   Invoke-WebRequest -Uri $Uri -OutFile '.\CreateOrUpdateAzAutoAccount.ps1'
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para establecer los valores de las variables que asignará a los parámetros de script:

   ```powershell
   $aadTenantId = (Get-AzContext).Tenant.Id
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   $suffix = Get-Random
   $automationAccountName = "az140-automation-51$suffix"
   $workspaceName = "az140-workspace-51$suffix"
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear el grupo de recursos que utilizará en este laboratorio:

   ```powershell
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear el área de trabajo de Azure Log Analytics que utilizará en este laboratorio:

   ```powershell
   New-AzOperationalInsightsWorkspace -Location $location -Name $workspaceName -ResourceGroupName $resourceGroupName
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, seleccione el archivo del menú superior, abra el script **C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzAutoAccount.ps1** e introduzca el código entre las líneas **82** y **86** en el comentario de múltiples líneas y guárdelo con el aspecto siguiente:

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
    throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, abra una nueva pestaña en el panel de scripts de **Administrador: Windows PowerShell ISE**, pegue el script siguiente y ejecútelo para crear la cuenta de Azure Automation que forma parte de la solución de escalado automático:

   ```powershell
   $Params = @{
     "AADTenantId" = $aadTenantId
     "SubscriptionId" = $subscriptionId 
     "UseARMAPI" = $true
     "ResourceGroupName" = $resourceGroupName
     "AutomationAccountName" = $automationAccountName
     "Location" = $location
     "WorkspaceName" = $workspaceName
   }

   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   .\CreateOrUpdateAzAutoAccount.ps1 @Params
   ```

   >**Nota**: Espere a que se complete el script. Esto puede tardar unos 10 minutos.

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en el panel de scripts de **Administrador: Windows PowerShell ISE**, revise la salida del script. 

   >**Nota**: La salida incluye un URI de webhook, el identificador del área de trabajo de Log Analytics y los valores de clave principal correspondientes que debe proporcionar al aprovisionar la aplicación de Azure Logic que forma parte de la solución de escalado automático. 
   
   >**Nota**: Registre el valor del URI de webhook. Lo necesitará más adelante en este laboratorio.

1. Para comprobar la configuración de la cuenta de Azure Automation, en la sesión de Escritorio remoto a **az140-dc-vm11**, inicie Microsoft Edge y vaya a [Azure Portal](https://portal.azure.com). Si se le solicita, inicie sesión con las credenciales de Azure AD de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Cuentas de Automation** y, en la hoja **Cuentas de Automation**, seleccione la entrada que representa la cuenta de Azure Automation recién aprovisionada (con el nombre que empieza por el prefijo **az140-automation-51**).
1. En la hoja Cuenta de Automation, en el menú vertical del lado izquierdo, en la sección **Automatización de procesos**, seleccione **Runbooks** y, en la lista de runbooks, compruebe la presencia del runbook **WVDAutoScaleRunbookARMBased**.
1. En la hoja Cuenta de Automation, en el menú vertical del lado izquierdo, en la sección **Configuración de la cuenta**, seleccione **Cuentas de ejecución** y, en la lista de cuentas de la derecha, junto a **+ Cuenta de ejecución de Azure**, haga clic en **Crear**.
1. En la hoja **Agregar cuenta de ejecución de Azure**, haga clic en **Crear** y compruebe que la cuenta nueva se creó correctamente.

#### <a name="task-3-create-an-azure-logic-app"></a>Tarea 3: Crear una aplicación de Azure Logic

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, cambie a la ventana **Administrador: Windows PowerShell ISE** y, en el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para descargar el script de PowerShell que utilizará para crear la aplicación de Azure Logic que forma parte de la solución de escalado automático:

   ```powershell
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   Set-Location -Path $labFilesfolder
   $uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
   Invoke-WebRequest -Uri $uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, seleccione el **Archivo** del menú superior y abra el script **C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzLogicApp.ps1**, introduzca el código entre las líneas **134** y **138** en el comentario de múltiples líneas y guárdelo con el aspecto siguiente:

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
    throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para establecer los valores de las variables que quiere asignar a los parámetros de script (sustituya el marcador de posición `<webhook_URI>` por el valor del URI de webhook que anotó anteriormente en este laboratorio):

   ```powershell
   $AADTenantId = (Get-AzContext).Tenant.Id
   $AzSubscription = (Get-AzContext).Subscription.Id
   $ResourceGroup = Get-AzResourceGroup -Name 'az140-51-RG'
   $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" -Name 'az140-21-hp1'
   $LogAnalyticsWorkspace = (Get-AzOperationalInsightsWorkspace -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $LogAnalyticsWorkspaceId = $LogAnalyticsWorkspace.CustomerId
   $LogAnalyticsWorkspaceKeys = (Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName $ResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspace.Name)
   $LogAnalyticsPrimaryKey = $LogAnalyticsWorkspaceKeys.PrimarySharedKey
   $RecurrenceInterval = 2
   $BeginPeakTime = '1:00'
   $EndPeakTime = '1:01'
   $TimeDifference = '0:00'
   $SessionThresholdPerCPU = 1
   $MinimumNumberOfRDSH = 1
   $MaintenanceTagName = 'CustomMaintenance'
   $LimitSecondsToForceLogOffUser = 5
   $LogOffMessageTitle = 'Autoscaling'
   $LogOffMessageBody = 'Forcing logoff due to autoscaling'

   $AutoAccount = (Get-AzAutomationAccount -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

   $WebhookURIAutoVar = '<webhook_URI>'
   ```

   >**Nota**: Los valores de los parámetros están orientados a acelerar el comportamiento del escalado automático. En el entorno de producción, debe ajustarlos para que coincidan con sus requisitos específicos.

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear la aplicación de Azure Logic que forma parte de la solución de escalado automático:

   ```powershell
   $Params = @{
     "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
     "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
     "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
     "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
     "UseARMAPI"                     = $true
     "HostPoolName"                  = $WVDHostPool.Name
     "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
     "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
     "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
     "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
     "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
     "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
     "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
     "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
     "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
     "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
     "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
     "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
     "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shut down."
     "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
     "WebhookURI"                    = $WebhookURIAutoVar
   }

   .\CreateOrUpdateAzLogicApp.ps1 @Params
   ```

   >**Nota**: Espere a que se complete el script. Esto puede tardar unos 2 minutos.

1. Para comprobar la configuración de la aplicación de Azure Logic, en la sesión de Escritorio remoto a **az140-dc-vm11**, cambie a la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Logic Apps** y, en la hoja **Logic Apps**, seleccione la entrada que representa la aplicación de Azure Logic recién aprovisionada denominada **az140-21-hp1_Autoscale_Scheduler**.
1. En la hoja **az140-21-hp1_Autoscale_Scheduler**, en el menú vertical del lado izquierdo, en la sección **Herramientas de desarrollo**, seleccione **Diseñador de aplicaciones lógicas**. 
1. En el panel del diseñador, haga clic en el rectángulo con la etiqueta **Periodicidad** y tenga en cuenta que puede usarlo para controlar la frecuencia con la que se evalúa la necesidad del escalado automático. 

### <a name="exercise-2-verify-and-review-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Ejercicio 2: Verificación y revisión del escalado automático de hosts de sesión de Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Verificar el escalado automático de hosts de sesión de Azure Virtual Desktop
1. Usar Azure Log Analytics para realizar un seguimiento de los eventos de Azure Virtual Desktop

#### <a name="task-1-verify-autoscaling-of-azure-virtual-desktop-session-hosts"></a>Tarea 1: Verificar el escalado automático de hosts de sesión de Azure Virtual Desktop

1. Para comprobar el escalado automático de los hosts de sesión de Azure Virtual Desktop, en la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, revise el estado de las tres máquinas virtuales de Azure en el grupo de recursos **az140-21-RG**.
1. Compruebe que dos de las tres máquinas virtuales de Azure están en proceso de desasignarse o que ya están **detenidas (desasignadas)** .

   >**Nota**: En cuanto compruebe que el escalado automático funciona, debe deshabilitar la aplicación de Azure Logic para minimizar los cargos correspondientes.

1. Para deshabilitar la aplicación de Azure Logic, en la sesión de Escritorio remoto a **az140-dc-vm11**, cambie a la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Logic Apps** y, en la hoja **Logic Apps**, seleccione la entrada que representa la aplicación de Azure Logic recién aprovisionada denominada **az140-21-hp1_Autoscale_Scheduler**.
1. En la hoja **az140-21-hp1_Autoscale_Scheduler**, en la barra de herramientas, haga clic en **Deshabilitar**. 
1. En la hoja **az140-21-hp1_Autoscale_Scheduler**, en la sección **Essentials**, revise la información, incluido el número de ejecuciones correctas en las últimas 24 horas y la sección **Resumen** que proporciona la frecuencia de periodicidad. 
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Cuentas de Automation** y, en la hoja **Cuentas de Automation**, seleccione la entrada que representa la cuenta de Azure Automation recién aprovisionada (con el nombre que empieza por el prefijo **az140-automation-51**).
1. En la hoja **Cuenta de Automation**, en el menú vertical del lado izquierdo, en la sección **Automatización de procesos**, seleccione **Trabajos** y revise la lista de trabajos correspondientes a invocaciones individuales del runbook **WVDAutoScaleRunbookARMBased**.
1. Seleccione el trabajo más reciente y, en su hoja, haga clic en el encabezado de pestaña **Todos los registros**. Se mostrará una lista detallada de los pasos de ejecución del trabajo.

#### <a name="task-2-use-azure-log-analytics-to-track-azure-virtual-desktop-events"></a>Tarea 2: Usar Azure Log Analytics para realizar un seguimiento de los eventos de Azure Virtual Desktop

>**Nota**: Puede usar Log Analytics para analizar el escalado automático y cualquier otro evento de Azure Virtual Desktop.

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Áreas de trabajo de Log Analytics** y, en la hoja **Áreas de trabajo de Log Analytics**, seleccione la entrada que representa el área de trabajo de Azure Log Analytics usada en este laboratorio (cuyo nombre comienza con el prefijo **az140-workspace-51**).
1. En la hoja Área de trabajo de Log Analytics, en el menú vertical del lado izquierdo, en la sección **General**, haga clic en **Registros** y, si es necesario, cierre la ventana **Le damos la bienvenida a Log Analytics** y vaya al panel **Consulta**.
1. En el panel **Consultas**, en el menú vertical **Todas las consultas** del lado izquierdo, seleccione **Azure Virtual Desktop** y revise las consultas predefinidas.
1. Cierre el panel **Consultas**. Esto mostrará automáticamente la pestaña **Nueva consulta 1**.
1. En la ventana de consulta, pegue la consulta siguiente y haga clic en **Ejecutar** para mostrar todos los eventos del grupo de hosts usado en este laboratorio:

   ```kql
   WVDTenantScale_CL
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

   >**Nota**: Si había un carácter de canalización adicional (|) en la segunda línea al usar la construcción cortada y pegada, quítelo para evitar un error. Esto podría aplicarse a cada consulta.
   >**Nota**: Si no ve ningún resultado, espere unos minutos e inténtelo de nuevo.

1. En la ventana de consulta, pegue la consulta siguiente, haga clic en **Ejecutar** para mostrar el número total de hosts de sesión actualmente en ejecución y de sesiones de usuario activas en el grupo de hosts de destino:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Number of running session hosts:"
     or logmessage_s contains "Number of user sessions:"
     or logmessage_s contains "Number of user sessions per Core:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. En la ventana de consulta, pegue la consulta siguiente y haga clic en **Ejecutar** para mostrar el estado de todas las máquinas virtuales del host de sesión en un grupo de hosts:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Session host:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. En la ventana de consulta, pegue la consulta siguiente y haga clic en **Ejecutar** para mostrar los errores y advertencias relacionados con el escalado:

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

>**Nota**: Omita el mensaje de error relacionado con `TenantId`

### <a name="exercise-3-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>Ejercicio 3: Detención y desasignación de máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detener y desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignará las máquinas virtuales de Azure aprovisionadas en este laboratorio para minimizar los cargos de proceso correspondientes.

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>Tarea 1: Desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para enumerar todas las máquinas virtuales de Azure creadas en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Desde la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para detener y desasignar todas las máquinas virtuales de Azure que creó en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.
