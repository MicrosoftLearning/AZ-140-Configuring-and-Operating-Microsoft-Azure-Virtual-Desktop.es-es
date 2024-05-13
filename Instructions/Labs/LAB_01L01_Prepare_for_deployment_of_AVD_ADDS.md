---
lab:
  title: "Laboratorio: Preparación de una implementación de Azure Virtual Desktop (AD\_DS)"
  module: 'Module 1: Plan an AVD Architecture'
---

# Laboratorio: Preparación de una implementación de Azure Virtual Desktop (AD DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta de Microsoft o una cuenta de Microsoft Entra con los roles de Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol de Administrador global en el inquilino de Microsoft Entra asociado a esa suscripción de Azure.

## Tiempo estimado

60 minutos

## Escenario del laboratorio

Debe prepararse para la implementación de un entorno de Active Directory Domain Services (AD DS).

## Objetivos
  
Después de completar este laboratorio, podrá:

- Implementación de un bosque de dominio único de Active Directory Domain Services (AD DS) mediante máquinas virtuales de Azure
- Integración de un bosque de AD DS en un inquilino de Microsoft Entra

## Archivos de laboratorio

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## Instrucciones

### Ejercicio 0: Incremento del número de cuotas de vCPU

Las tareas principales de este ejercicio son las siguientes:

1. Averiguar el uso actual de vCPU
1. Solicitar un aumento de la cuota de vCPU

#### Tarea 1: Averiguar el uso actual de vCPU

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. Seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda en Azure Portal para abrir el panel de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **PowerShell**. 

   >**Nota**: Si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que utiliza en este laboratorio y seleccione **Crear almacenamiento**. 

1. En Azure Portal, en la sesión de PowerShell de **Cloud Shell**, ejecute lo siguiente para registrar los proveedores de recursos **Microsoft.Compute** y **Microsoft.Network**, siempre que no estén registrados:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. En Azure Portal, en la sesión de PowerShell de **Cloud Shell**, ejecute lo siguiente para comprobar el estado de registro del proveedor de recursos **Microsoft.Compute**:

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Nota**: Compruebe que el estado aparece como **Registrado**. Si no es así, espere unos minutos y repita este paso.

1. En Azure Portal, en la sesión de PowerShell de **Cloud Shell**, ejecute lo siguiente para establecer la ubicación de los siguientes comandos (reemplace el marcador de posición `<Azure_region>` por el nombre de la región de Azure que quiere usar para este laboratorio, como, por ejemplo, por ejemplo, `eastus`):

   ```powershell
   $location = '<Azure_region>'
   ```

1. En Azure Portal, en la sesión de PowerShell de **Cloud Shell**, ejecute lo siguiente para identificar el uso actual de las CPU virtuales y los límites correspondientes para las máquinas virtuales de Azure **StandardDSv3Family** y **StandardBSFamily**: 

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

   > **Nota**: Para identificar los nombres de las regiones de Azure, en **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute `(Get-AzLocation).Location`.
   
1. Examine la salida del comando ejecutado en el paso anterior y asegúrese de que tiene al menos **30** CPU virtuales disponibles en las CPU virtuales de la **familia DSv3 estándar** de las máquinas virtuales de Azure en la región de Azure de destino. Si esto ya es así, avance directamente al siguiente ejercicio. Si no, prosiga con la siguiente tarea de este ejercicio. 

#### Tarea 2: Solicitar un aumento de la cuota de vCPU

1. En Azure Portal, busque y seleccione **Suscripciones** y, en la hoja **Suscripciones**, seleccione la entrada que representa la suscripción de Azure que desea usar en este laboratorio.
1. En Azure Portal, en la hoja Suscripciones, en la sección **Configuración** del menú vertical de la izquierda, seleccione **Uso y cuotas**. 

   **Nota:** Es posible que no tenga que generar una incidencia de soporte técnico para aumentar las cuotas.

   **Nota:** La solicitud de aumento de cuota requiere que se inicie de sesión con autenticación multifactor (MFA). Si necesita configurar su cuenta con MFA, consulte [Planear una implementación de autenticación multifactor de Azure Active Directory](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted). 
   
1. En la hoja **Pase para Azure: Patrocinio | Uso y cuotas**, seleccione **Región**, en la lista desplegable, seleccione la casilla situada junto al nombre de la región de Azure que tiene intención de usar para este laboratorio, seleccione **Aplicar**asegúrese de que la entrada **Proceso** aparece en la lista desplegable que está a la izquierda de la entrada **Región** y, en el cuadro de búsqueda, escriba **DSv3 estándar**. 
1. En la lista de resultados, seleccione la casilla situada junto al elemento **CPU virtuales de la familia DSv3 estándar**, seleccione la entrada **Solicitar aumento de cuota** en la barra de herramientas y, en la lista desplegable, seleccione **Escribir un nuevo límite**.
1. En el panel **Solicitar un aumento de la cuota**, en el cuadro de texto de la columna **Nuevo límite**, escriba **30** y, después, seleccione **Enviar**.
1. Si se le solicita, en el panel **Solicitar aumento de cuota**, seleccione **Autenticar con autenticación multifactor** y siga las indicaciones para autenticarse.
1. Deje que la solicitud de cuota se complete.  Transcurrido un momento, la hoja **Detalles de la cuota** reflejará que la solicitud se ha aprobado y que la cuota ha aumentado. Cierre la hoja **Detalles de la cuota**.

   >**Nota**: En función de la elección de la región de Azure y la demanda actual, es posible que sea necesario generar una solicitud de soporte técnico. Para obtener instrucciones sobre el proceso de creación de una solicitud de soporte técnico, consulte [Creación de una solicitud de soporte técnico de Azure](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Ejercicio 1: Implementación de un dominio de Active Directory Domain Services (AD DS).

Las tareas principales de este ejercicio son las siguientes:

1. Preparar una implementación de máquina virtual de Azure
1. Implementar una máquina virtual de Azure que ejecuta un controlador de dominio de AD DS mediante una plantilla de inicio rápido de Azure Resource Manager
1. Implementar una máquina virtual de Azure que ejecuta Windows 10 mediante una plantilla de inicio rápido de Azure Resource Manager
1. Implementación de Azure Bastion

#### Tarea 1: Preparar una implementación de máquina virtual de Azure

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En el portal Azure, utilice el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página del portal Azure para buscar y navegar hasta el panel de **Microsoft Entra ID**.
1. En el panel de **Información general** del arrendatario Microsoft Entra y, en el menú vertical de la izquierda, en la sección **Administrar**, haga clic en **Propiedades**.
1. En la parte inferior de la hoja **Propiedades** del inquilino de Microsoft Entra, seleccione el vínculo **Administrar valores predeterminados de seguridad**.
1. En el panel **Habilitar valores predeterminados de seguridad**, si es necesario, seleccione **Deshabilitado (no recomendado)**, seleccione el botón de opción **Mi organización tiene previsto utilizar el Acceso condicional** y seleccione **Guardar**, y luego, seleccione **Deshabilitar**.
1. En Azure Portal, seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda para abrir el panel de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **PowerShell**. 

   >**Nota**: Si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que utiliza en este laboratorio y seleccione **Crear almacenamiento**. 


#### Tarea 2: Implementar una máquina virtual de Azure que ejecuta un controlador de dominio de AD DS mediante una plantilla de inicio rápido de Azure Resource Manager

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, desde la sesión de PowerShell en el panel Cloud Shell, ejecute lo siguiente para crear un grupo de recursos (reemplace el marcador de posición `<Azure_region>` por el nombre de la región de Azure que quiere usar para este laboratorio; por ejemplo, `eastus`):

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. En Azure Portal, cierre el panel **Cloud Shell**.
1. En el equipo de laboratorio, en la misma ventana del explorador web, abra otra pestaña del explorador web y vaya a una versión personalizada de la plantilla de inicio rápido denominada [Creación de una nueva máquina virtual de Windows y creación de un nuevo bosque, un dominio y un controlador de dominio de AD](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain). 
1. En la página **Crear una nueva máquina virtual de Windows y crear un nuevo bosque, dominio y DC de AD**, desplácese hacia abajo en la página y seleccione **Implementar en Azure**. Se redirigirá automáticamente el explorador a la hoja **Create an Azure VM with a new AD Forest** (Crear una VM de Azure con un nuevo bosque de AD) en Azure Portal.
1. En la hoja **Create an Azure VM with a new AD Forest** (Crear una máquina virtual de Azure con un nuevo bosque de AD), seleccione **Editar parámetros**.
1. En la hoja **Editar parámetros**, seleccione **Cargar archivo**; En el cuadro de diálogo **Abrir**, seleccione **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**, **Abrir** y, luego, **Guardar**. 
1. En la hoja **Create an Azure VM with a new AD Forest** (Crear una VM de Azure con un nuevo bosque de AD), especifique la configuración siguiente (deje las demás opciones con los valores existentes):

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az140-11-RG**|
   |Nombre de dominio|**adatum.com**|

1. En la hoja **Create an Azure VM with a new AD Forest** (Crear una máquina virtual de Azure con un nuevo bosque de AD), seleccione **Revisar y crear** y luego **Crear**.

   > **Nota**: Espere a que la implementación se complete antes de avanzar al siguiente ejercicio. La implementación puede tardar unos 20-25 minutos. 

#### Tarea 3: Implementar una máquina virtual de Azure que ejecuta Windows 10 mediante una plantilla de inicio rápido de Azure Resource Manager

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, abra una sesión de PowerShell del panel de Cloud Shell y ejecute lo siguiente para agregar una subred denominada **cl-Subnet** a la red virtual **az140-aadds-vnet11** que creamos en la tarea anterior:

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. En Azure Portal, en la barra de herramientas del panel de Cloud Shell, seleccione el icono **Cargar/Descargar archivos**; en el menú desplegable, seleccione **Cargar** y cargue los archivos **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json** y **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json** en el directorio principal de Cloud Shell.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para implementar una máquina virtual de Azure que ejecute Windows 10, la cual servirá como cliente en la subred recientemente creada:

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > **Nota**: No espere a que se completen las implementaciones, sino que avance a la siguiente tarea. La implementación puede tardar unos 10 minutos.

#### Tarea 4: Implementación de Azure Bastion 

> **Nota**: Azure Bastion permite la conexión a las máquinas virtuales de Azure sin puntos de conexión públicos que implementamos en la tarea anterior de este ejercicio, al tiempo que proporciona protección contra vulnerabilidades de seguridad de fuerza bruta que tienen como destino las credenciales de nivel de sistema operativo.

> **Nota**: Asegúrese de que el explorador tiene habilitada la función emergente.

1. En la ventana del explorador en que se muestra Azure Portal, abra otra pestaña y, en la pestaña del explorador, vaya a [Azure Portal](https://portal.azure.com).
1. En Azure Portal, seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda para abrir el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para agregar una subred denominada **AzureBastionSubnet** a la red virtual **az140-adds-vnet11** que creamos anteriormente en este ejercicio:

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cierre el panel de Cloud Shell.
1. En Azure Portal, busque y seleccione **Bastions** y, en la hoja **Bastions**, seleccione **+ Crear**.
1. En la pestaña **Básico** de la hoja **Crear una instancia de Bastion**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az140-11-RG**|
   |Nombre|**az140-11-bastion**|
   |Region|la misma región de Azure en la que se han implementado los recursos en las tareas anteriores de este ejercicio.|
   |Nivel|**Basic**|
   |Red virtual|**az140-adds-vnet11**|
   |Subnet|**AzureBastionSubnet (10.0.254.0/24)**|
   |Dirección IP pública|**Crear nuevo**|
   |Nombre de la IP pública|**az140-adds-vnet11-ip**|

1. En la pestaña **Revisar y crear** de la hoja **Crear una instancia de Bastion**, seleccione **Crear**:

   > **Nota**: Espere a que la implementación se complete antes de avanzar al siguiente ejercicio. La implementación puede tardar unos 10 minutos.

### Ejercicio 2: Integración de un bosque de AD DS en un inquilino de Microsoft Entra
  
Las tareas principales de este ejercicio son las siguientes:

1. Creación de usuarios y grupos de AD DS que se sincronizarán con Microsoft Entra
1. Configurar el sufijo UPN de AD DS
1. Creación de un usuario de Microsoft Entra que se usará para configurar la sincronización con Microsoft Entra
1. Instalación de Microsoft Entra Connect

#### Tarea 1: Creación de usuarios y grupos de AD DS que se sincronizarán con Microsoft Entra

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione **az140-dc-vm11**.
1. En el panel **az140-dc-vm11**, seleccione **Conectar**, en el menú desplegable, seleccione **Conectar a través de Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Bastion a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para deshabilitar la seguridad mejorada de Internet Explorer para administradores:

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear una unidad organizativa de AD DS que contendrá objetos incluidos en el ámbito de sincronización en el inquilino de Azure AD usado en este laboratorio:

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear una unidad organizativa de AD DS que contendrá objetos de equipo de equipos cliente unidos a un dominio de Windows 10:

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear cuentas de usuario de AD DS que se sincronizarán con el inquilino de Microsoft Entra usado en este laboratorio (reemplace los dos marcadores de posición `<password>` por una contraseña aleatoria y compleja):

   > **Nota**: Asegúrese de registrar las contraseñas usadas, ya que las necesitará más adelante tanto en este laboratorio como en los siguientes.

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **Nota**: El script crea nueve cuentas de usuario sin privilegios llamadas **aduser1** - **aduser9** y una cuenta con privilegios que es miembro del grupo **ADATUM\\Administradores de dominio** llamado **wvdadmin1**.

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear un objeto de grupo de AD DS que se sincronizará con el inquilino de Microsoft Entra usado en este laboratorio:

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. Desde la consola **Administrador: Windows PowerShell consola ISE**, ejecute lo siguiente para agregar miembros a los grupos que ha creado en el paso anterior:

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### Tarea 2: Configurar el sufijo UPN de AD DS

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar la última versión del módulo PowerShellGet (seleccione **Sí** cuando se le solicite confirmación):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar la última versión del módulo Az PowerShell (seleccione **Sí a todo** cuando se le solicite confirmación):

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

   > **Nota**: Es posible que tenga que esperar de 3 a 5 minutos antes de que aparezca cualquier salida de la instalación del módulo Az. También es posible que tenga que esperar otros 5 minutos **después** de que se haya detenido la salida. Este es el comportamiento esperado.

1. Desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para iniciar sesión en la suscripción de Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Cuando se le solicite, proporcione las credenciales de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para recuperar la propiedad Id del inquilino de Microsoft Entra asociado a la suscripción de Azure:

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar e importar la última versión del módulo PowerShell de Azure AD:

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para autenticarse en el inquilino de Microsoft Entra:

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. Cuando se le solicite, inicie sesión con las mismas credenciales que usó anteriormente en esta tarea (la cuenta de usuario con el rol de Propietario en la suscripción que usa en este laboratorio). 
1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para recuperar el nombre de dominio DNS principal del inquilino de Microsoft Entra asociado a la suscripción de Azure:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para agregar el nombre de dominio DNS principal del inquilino de Microsoft Entra asociado a la suscripción de Azure a la lista de sufijos de UPN del bosque de AD DS:

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente a fin de asignar el nombre de dominio DNS principal del inquilino de Microsoft Entra asociado a la suscripción de Azure como el sufijo de UPN para todos los usuarios del dominio de AD DS:

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para asignar el sufijo de UPN **adatum.com** al usuario de dominio **Alumno**:

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### Tarea 3: Crear un usuario de Microsoft Entra que se utilizará para configurar la sincronización de directorios

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear un usuario de Microsoft Entra (reemplace el marcador de posición `<password>` por una contraseña aleatoria y compleja):

   > **Nota**: Asegúrese de registrar la contraseña que ha usado. **Lo necesitará más adelante en este laboratorio y en los siguientes.**.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para asignar el rol de Administrador global al usuario de Microsoft Entra recién creado: 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para conocer el nombre principal del usuario de Microsoft Entra recién creado:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **Nota**: Registre el nombre de usuario principal **y** la contraseña. ya que lo necesitará más adelante en este ejercicio. 


#### Tarea 4: Instalar Microsoft Entra Connect

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para habilitar TLS 1.2:

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. Dentro de la sesión de Bastion a **az140-dc-vm11**, inicie Internet Explorer y navegue hasta la [página de descarga de Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download).
1. En la [página de descarga de Microsoft Edge para empresas](https://www.microsoft.com/en-us/edge/business/download), descargue la versión estable más reciente de Microsoft Edge, instálela, iníciela y configúrela con las opciones predeterminadas.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, use Microsoft Edge para ir a [Azure portal](https://portal.azure.com). Si se le solicita, utilice las credenciales de Microsoft Entra de la cuenta de usuario con el rol de Propietario para iniciar sesión en la suscripción que usa en este laboratorio.
1. En el portal Azure, utilice el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página del portal Azure para buscar y navegar hasta el panel **Microsoft Entra ID** y, en su panel de inquilino de Microsoft Entra, en la sección **Administrar** del menú central, seleccione **Microsoft Entra Connect**.
1. En el panel de **Microsoft Entra Connect**, seleccione el vínculo **Programador de sincronización** en el menú de servicio, y luego, seleccione el vínculo **Descargar Microsoft Entra Connect**. Se abrirá automáticamente una nueva pestaña del navegador en la que se mostrará la página de descarga de **Microsoft Entra Connect**.
1. En la página de descarga de **Microsoft Entra Connect** seleccione** Descargar**.
1. Si se le pide que ejecute o guarde el instalador **AzureADConnect.msi**, seleccione **Ejecutar**. De lo contrario, abra el archivo después de descargarlo para iniciar el asistente de **Microsoft Azure Active Directory Connect**.
1. En la página **Bienvenido a Azure AD Connect** del asistente de **Microsoft Azure Active Directory Connect**, seleccione la casilla **I agree to the license terms and privacy notice** (Acepto los términos de licencia y el aviso de privacidad) y luego en **Continuar**.
1. En la página **Configuración rápida** del asistente de **Microsoft Azure Active Directory Connect**, seleccione la opción **Personalizar**.
1. En la página **Instalar componentes necesarios**, deje desactivadas todas las opciones de configuración opcionales y seleccione **Instalar**.
1. En la página **Inicio de sesión de usuario**, asegúrese de que solo esté habilitada la opción **Sincronización de hash de contraseñas** y seleccione **Siguiente**.
1. En la página **Conectar con Azure AD**, autentíquese con las credenciales de la cuenta de usuario **aadsyncuser** que creó en el ejercicio anterior y seleccione **Siguiente**. 

   > **Nota**: Proporcione el atributo userPrincipalName de la cuenta **aadsyncuser** que registró anteriormente en este ejercicio y especifique la contraseña que estableció anteriormente en este laboratorio como contraseña.

1. En la página **Conectar sus directorios**, seleccione el botón **Agregar directorio** situado a la derecha de la entrada del bosque **adatum.com**.
1. En la ventana **Cuenta del bosque de AD**, asegúrese de que está seleccionada la opción **Crear una cuenta de AD**, especifique las credenciales siguientes y seleccione **Aceptar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**ADATUM\Student**|
   |Contraseña|**Pa55w.rd1234**|

1. De nuevo en la página **Conectar sus directorios**, asegúrese de que la entrada **adatum.com** aparece como un directorio configurado y seleccione **Siguiente**.
1. En la página **Configuración de inicio de sesión de Azure AD**, observe la advertencia que indica **que los usuarios no podrán iniciar sesión en Azure AD con credenciales locales si el sufijo UPN no coincide con un nombre de dominio comprobado**, active la casilla **Continue without matching all UPN suffixes to verified domain** (Continuar sin hacer coincidir todos los sufijos UPN con el dominio comprobado) y seleccione **Siguiente**.

   > **Nota**: Esto es algo esperable, ya que el inquilino de Microsoft Entra no tiene un dominio DNS personalizado comprobado que coincida con uno de los sufijos UPN del AD DS **adatum.com**.

1. En la página **Filtrado de dominios y unidades organizativas**, seleccione la opción **Sincronizar los dominios y las unidades organizativas seleccionados**, desactive todas las casillas, seleccione solo la casilla situada junto a la unidad organizativa **ToSync** y **Siguiente**.
1. En la página **Identificación de forma exclusiva de usuarios**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la página **Filtrar usuarios y dispositivos,** acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la página **Características opcionales**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la página **Listo para configurar**, asegúrese de que la casilla **Start the synchronization process when configuration completes** (Iniciar el proceso de sincronización cuando se complete la configuración) esté activada y seleccione **Instalar**.

   > **Nota**: La instalación debería durar unos 5 minutos.

1. Revise la información de la página **Configuración completada** y seleccione **Salir** para cerrar la ventana **Microsoft Azure Active Directory Connect**.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra Azure Portal, vaya a la hoja **Usuarios - Todos los usuarios** del inquilino de Microsoft Entra del laboratorio de Adatum.
1. En la hoja **Usuarios \| Todos los usuarios**, tenga en cuenta que la lista de objetos de usuario incluye la lista de cuentas de usuario de AD DS que creó anteriormente en este laboratorio, con la entrada **Sí** que aparece en la columna **Sincronización local habilitada**.

   > **Nota**: Es posible que tenga que esperar unos minutos y actualizar la página del explorador para que aparezcan las cuentas de usuario de AD DS.
