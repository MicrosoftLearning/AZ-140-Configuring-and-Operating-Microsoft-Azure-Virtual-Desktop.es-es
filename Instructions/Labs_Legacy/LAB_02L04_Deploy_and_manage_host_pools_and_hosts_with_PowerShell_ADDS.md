---
lab:
  title: "Laboratorio: Implementación y administración de hosts y grupos de hosts mediante PowerShell (AD\_DS)"
  module: 'Module 2: Implement a WVD Infrastructure'
---

# Laboratorio: Implementación y administración de hosts y grupos de hosts mediante PowerShell
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Azure AD con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Azure AD asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**

## Tiempo estimado

60 minutos

## Escenario del laboratorio

Debe automatizar la implementación de grupos de hosts y hosts de Azure Virtual Desktop mediante PowerShell en un entorno de Active Directory Domain Services (AD DS).

## Objetivos
  
Después de completar este laboratorio, podrá:

- Implementar grupos de hosts y hosts de Azure Virtual Desktop mediante PowerShell
- Agregar hosts al grupo de hosts de Azure Virtual Desktop mediante PowerShell

## Archivos de laboratorio

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## Instrucciones

### Ejercicio 1: Implementación de grupos de hosts y hosts de sesión de Azure Virtual Desktop mediante PowerShell
  
Las tareas principales de este ejercicio son las siguientes:

1. Preparación para la implementación del grupo de hosts de Azure Virtual Desktop mediante PowerShell
1. Creación de un grupo de hosts de Azure Virtual Desktop mediante PowerShell
1. Realice una implementación basada en plantillas de una máquina virtual Azure que ejecute Windows 11 Enterprise mediante PowerShell
1. Agregue una Azure VM que ejecute Windows 11 Enterprise como host de sesión al grupo de hosts Azure Virtual Desktop mediante PowerShell
1. Comprobación de la implementación del host de sesión de Azure Virtual Desktop

#### Tarea 1: Preparación para la implementación del grupo de hosts de Azure Virtual Desktop mediante PowerShell

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, haga clic en **az140-dc-vm11**.
1. En el panel **az140-dc-vm11**, seleccione **Conectar**, en el menú desplegable, seleccione **Conectar a través de Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Bastion a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar el módu.lo DesktopVirtualization de PowerShell (cuando se le solicite, haga clic en **Sí a todo**):

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **Nota**: Ignore las advertencias relativas a los módulos de PowerShell existentes en uso.

1. En la sesión de Bastion a **az140-dc-vm11**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Si se le solicita, inicie sesión con las credenciales de Azure AD de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Bastion a **az140-dc-vm11**, en Azure Portal, utilice el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a **Redes virtuales** y, en la hoja **Redes virtuales**, seleccione **az140-adds-vnet11**. 
1. En la hoja **az140-adds-vnet11**, seleccione **Subredes**, y en la hoja **Subredes**, seleccione **+ Subred**. Después, en la hoja **Agregar subred**, especifique la siguiente configuración (deje todas las demás opciones con sus valores predeterminados) y haga clic en **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Nombre|**hp3-Subnet**|
   |Intervalo de direcciones de subred|**10.0.3.0/24**|

#### Tarea 2: Creación de un grupo de hosts de Azure Virtual Desktop mediante PowerShell

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para identificar la región de Azure que hospeda la red virtual de Azure **az140-adds-vnet11**:

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear un grupo de recursos que hospedará el grupo de hosts y sus recursos:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear un grupo de hosts vacío:

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **Nota**: El cmdlet **New-AzWvdHostPool** permite crear un grupo de hosts, un área de trabajo y el grupo de aplicaciones de escritorio, así como registrar el grupo de aplicaciones de escritorio con el área de trabajo. Tiene la opción de crear un área de trabajo nueva o usar una existente.

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para recuperar el atributo objectID del grupo de Azure AD denominado **az140-wvd-pooled**:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para asignar el grupo de Azure AD denominado **az140-wvd-pooled** al grupo de aplicaciones de escritorio predeterminado del nuevo grupo de hosts creado:

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### Tarea 3: Realice una implementación basada en plantillas de una máquina virtual Azure que ejecute Windows 11 Enterprise mediante PowerShell

1. En el equipo de laboratorio, vaya a la cuenta de almacenamiento implementada. En la hoja Recurso compartido de archivos, seleccione el recurso compartido de archivos **az140-22-profiles**.

1. Seleccione **Cargar** y cargue los archivos de laboratorio **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json** y **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json** al recurso compartido de archivos.

1. En la sesión de Bastion a **az140-dc-vm11**, abra el Explorador de archivos y vaya a la unidad Z: configurada anteriormente o a la letra de unidad asignada a la conexión del recurso compartido de archivos. Copie los archivos de implementación cargados en **C:\AllFiles\Labs\02**.

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para implementar una máquina virtual Azure con Windows 11 Enterprise (multi sesión) que servirá como host de sesión de Azure Virtual Desktop en el grupo de hosts que creó en la tarea anterior:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **Nota**: Espere a que la implementación se complete antes de avanzar a la siguiente tarea. Puede tardar entre 5 y 10 minutos. 

   > **Nota**: La implementación usa una plantilla de Azure Resource Manager para aprovisionar una máquina virtual de Azure y aplica una extensión de máquina virtual que une automáticamente el sistema operativo al dominio AD DS **adatum.com**.

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para verificar que el tercer host de sesiones se unió correctamente al dominio AD DS **adatum.com**:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### Tarea 4: Agregue una máquina virtual Azure que ejecute Windows 11 Enterprise como host al grupo de hosts Azure Virtual Desktop mediante PowerShell

1. En la sesión de Bastion a **az140-dc-vm11**, en la ventana del explorador que muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, en la lista de máquinas virtuales, seleccione **az140-24-p3-0**.
1. En el panel **az140-24-p3-0**, seleccione **Conectar**, en el menú desplegable, seleccione **Conectar**
1. Asegúrese de que la **Conexión utilizando** dice **Dirección IP privada | 10.0.3.4**
1. En la sección **RDP nativo**, seleccione **Descargar archivo RDP**.
1. Si se le solicita, seleccione **Conservar**, y luego haga clic en el archivo **az140-24-p3-0.rdp** descargado.
1. Cuando se le solicite, inicie sesión con las siguientes credenciales:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**ADATUM\\Student**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-24-p3-0**, inicie **Windows PowerShell ISE** como administrador.
1. En la sesión de Escritorio remoto a **az140-24-p3-0**, desde el panel de scripts **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear una carpeta que hospedará archivos necesarios para agregar la máquina virtual de Azure recién implementada como host de sesiones al grupo de host que aprovisionó anteriormente en este laboratorio:

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

   >**Nota**: tenga cuidado al utilizar la construcción [T] para copiar los cmdlet de PowerShell. En algunos casos, el texto copiado puede ser incorrecto, como el signo $ que se muestra como un carácter de 4 números. Deberá corregirlos antes de emitir el cmdlet. Cópielos en el panel **Script** de PowerShell ISE, realice las correcciones allí y, a continuación, resalte el texto corregido y presione **F8** (**Ejecutar selección**).

1. En la sesión de Escritorio remoto a **az140-24-p3-0**, desde el panel de scripts **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para descargar el agente de Azure Virtual Desktop y los instaladores del cargador de arranque, necesarios para agregar el host de sesiones al grupo de hosts:

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. Desde la consola **Administrador: Consola Windows PowerShell ISE**, ejecute lo siguiente para instalar la última versión del módulo Az PowerShell, cuando se le solicite instalar el proveedor NuGet, pulse **Y**:

   ```powershell
   Install-Module -Name Az -Force
   ```

   > **Nota**: Es posible que tenga que esperar de 3 a 5 minutos antes de que aparezca cualquier salida de la instalación del módulo Az. También es posible que tenga que esperar otros 5 minutos **después** de que se haya detenido la salida. Este es el comportamiento esperado.

1. Desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para iniciar sesión en la suscripción de Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Cuando se le solicite, proporcione las credenciales de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Escritorio remoto a **az140-24-p3-0**, desde el panel de scripts **Administrador: En la consola Windows PowerShell ISE**, ejecute lo siguiente para generar el token necesario para unir este host de sesión al grupo que aprovisionó anteriormente en este ejercicio:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **Nota**: Se necesita un token de registro para autorizar a un host de sesión a unirse al grupo de hosts. El valor de la fecha de expiración del token debe estar entre una hora y un mes a partir de la fecha y hora actuales.

1. En la sesión de Escritorio remoto a **az140-24-p3-0**, desde el panel de scripts **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar el agente Azure Virtual Desktop:

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. En la sesión de Escritorio remoto a **az140-24-p3-0**, desde el panel de scripts **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar el cargador de arranque de Azure Virtual Desktop:

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

1. Dentro de la sesión de Escritorio remoto a **az140-24-p3-0**, haga clic con el botón derecho en **Inicio**, en el menú contextual, seleccione **Apagar o cerrar sesión** y, en el menú en cascada, haga clic en **Cerrar sesión**.

#### Tarea 5: Comprobación de la implementación del host de Azure Virtual Desktop

1. Cambie al equipo de laboratorio y, en el explorador web que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop**. En la hoja **Azure Virtual Desktop**, seleccione **Grupos de hosts** y, en la hoja **Grupos de hosts\| de Azure Virtual Desktop**, seleccione la entrada **az140-24-hp3** que representa el grupo recién modificado.
1. En la hoja **az140-24-hp3**, en el menú vertical del lado izquierdo, en la sección **Administrar**, haga clic en **Hosts de sesión**. 
1. En la hoja **Hosts de sesión de\| az140-24-hp3**, compruebe que la implementación incluye un único host.

#### Tarea 6: Administración de grupos de aplicaciones con PowerShell

1. En el equipo de laboratorio, cambie la sesión de Bastion a **az140-dc-vm11**, y desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear un grupo de Escritorio remoto:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para enumerar las aplicaciones de menú de **Inicio** en los grupos de host y revisar la salida:

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **Nota**: Para cualquier aplicación que quiera publicar, debe registrar la información incluida en la salida, incluidos parámetros como **FilePath**, **IconPath** e **IconIndex**.

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para publicar Microsoft Word:

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para publicar Microsoft Word:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. Cambie al equipo de laboratorio y, en el explorador web que muestra Azure Portal, en la hoja **Hosts de sesión de \|az140-24-hp3**, en el menú vertical del lado izquierdo, en la sección **Administrar**, seleccione **Grupos de aplicaciones**.
1. En la hoja **Grupos de aplicaciones de \|az140-24-hp3**, en la lista de grupos de aplicaciones, seleccione la entrada **az140-24-hp3-Office365-RAG**.
1. En la hoja **az140-24-hp3-Office365-RAG**, compruebe la configuración del grupo de aplicaciones, incluidas las aplicaciones y las asignaciones.

### Ejercicio 2: Detener y desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detención y desasignación de máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignará las máquinas virtuales de Azure aprovisionadas en este laboratorio para minimizar los cargos de proceso correspondientes.

#### Tarea 1: Desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para enumerar todas las máquinas virtuales de Azure creadas en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. Desde la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para detener y desasignar todas las máquinas virtuales de Azure que creó en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.
