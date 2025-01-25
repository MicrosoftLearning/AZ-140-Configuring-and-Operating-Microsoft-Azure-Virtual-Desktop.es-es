---
lab:
  title: "Laboratorio: preparación de una implementación de Azure Virtual Desktop (Microsoft\_Entra\_DS)"
  module: 'Module 1: Plan an AVD Architecture'
---

# Laboratorio: preparación de una implementación de Azure Virtual Desktop (Microsoft Entra DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure
- Una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure y con el rol Propietario o Colaborador en la suscripción de Azure

## Tiempo estimado

150 minutos

>**Nota**: El aprovisionamiento de una instancia de Microsoft Entra DS conlleva un tiempo de espera de aproximadamente 90 minutos.

## Escenario del laboratorio

Debe prepararse para la implementación de Azure Virtual Desktop en un entorno de Azure Active Directory Domain Services (Microsoft Entra DS)

## Objetivos
  
Después de completar este laboratorio, podrá:

- Implementación de un dominio de Microsoft Entra DS
- Configuración del entorno de dominio de Microsoft Entra DS

## Archivos de laboratorio

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

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

1. En Azure Portal, en la sesión de PowerShell de **Cloud Shell**, ejecute lo siguiente para registrar el proveedor de recursos **Microsoft.Compute**, en caso de que no esté registrado:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. En Azure Portal, en la sesión de PowerShell de **Cloud Shell**, ejecute lo siguiente para comprobar el estado de registro del proveedor de recursos **Microsoft.Compute**:

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Nota**: Compruebe que el estado aparece como **Registrado**. Si no es así, espere unos minutos y repita este paso.

1. En Azure Portal, en la sesión de PowerShell del **Cloud Shell**, ejecute lo siguiente para crear una variable de PowerShell con el nombre de una región de Azure (reemplace el marcador de posición `<Azure_region>` por el nombre de la región de Azure que quiere usar para este laboratorio, como, por ejemplo, por ejemplo, `eastus`):

   ```powershell
   $location = '<Azure_region>'
   ```

   > **Nota**: Para identificar los nombres de las regiones de Azure, en **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute `(Get-AzLocation).Location`.
   
1. En Azure Portal, en la sesión de PowerShell del **Cloud Shell**, ejecute lo siguiente para identificar el uso actual de las CPU virtuales y los límites correspondientes para las máquinas virtuales de Azure **StandardDSv3Family** :

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. Revise la salida del comando ejecutado en el paso anterior y asegúrese de que tiene al menos **20** CPU virtuales disponibles en la **familia DSv3 estándar** de las máquinas virtuales de Azure en la región de Azure de destino. Si esto ya es así, avance directamente al siguiente ejercicio. Si no, prosiga con la siguiente tarea de este ejercicio. 

#### Tarea 2: Solicitar un aumento de la cuota de vCPU

1. En Azure Portal, busque y seleccione **Suscripciones** y, en la hoja **Suscripciones**, seleccione la entrada que representa la suscripción de Azure que desea usar en este laboratorio.
1. En Azure Portal, en la hoja Suscripciones, en la sección **Configuración** del menú vertical de la izquierda, seleccione **Uso y cuotas**. 
1. En la hoja  **Pase para Azure - Patrocinio | Uso y cuotas** , selecciona las siguientes flechas desplegables en la barra de búsqueda superior:

   |**Configuración**|**Valor**|
   |---|---|
   |**Buscar**|**DSv3 estándar**|
   |**Todas las ubicaciones**|**Borrar todo** y, a continuación, compruebe *su ubicación*|
   |**Proveedor de recursos** | **Microsoft.Compute** |
   
1. En el elemento **CPU virtual de la Familia DSv3 estándar** devuelto, seleccione el icono de lápiz (**Editar**).
1. En la hoja **Detalles de la cuota**, en el cuadro de texto de la columna **Límite nuevo**, escriba **30** y, a continuación, seleccione **Guardar y continuar**.
1. Deje que la solicitud de cuota se complete.  Transcurrido un momento, la hoja **Detalles de la cuota** reflejará que la solicitud se ha aprobado y que la cuota ha aumentado. Cierre la hoja **Detalles de la cuota**.

    >**Nota**: En función de la elección de la región de Azure y la demanda actual, es posible que sea necesario generar una solicitud de soporte técnico. Para obtener instrucciones sobre el proceso de creación de una solicitud de soporte técnico, consulte [Creación de una solicitud de soporte técnico de Azure](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Ejercicio 1: Implementar un dominio de Active Directory Domain Services (AD DS)

Las tareas principales de este ejercicio son las siguientes:

1. Creación y configuración de una cuenta de usuario de Microsoft Entra para la administración del dominio de Microsoft Entra DS
1. Implementar una instancia de Microsoft Entra DS mediante Azure Portal
1. Configurar la red y la identidad de la implementación de Microsoft Entra DS

#### Tarea 1: Creación y configuración de una cuenta de usuario de Microsoft Entra para la administración del dominio de Microsoft Entra DS

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure.
1. En el explorador web donde se muestra Azure Portal, vaya a la hoja **Información general** del inquilino de Microsoft Entra y, en el menú vertical de la izquierda, en la sección **Administrar**, haga clic en **Propiedades**.
1. En la parte inferior de la hoja **Propiedades** del inquilino de Microsoft Entra, seleccione el vínculo **Administrar valores predeterminados de seguridad**.
1. Si es necesario, en la hoja **Activación de los valores predeterminados de seguridad**, seleccione **No**, active la casilla **Mi organización usa el acceso condicional** y seleccione **Guardar**.
1. En Azure Portal, seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda para abrir el panel de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **PowerShell**. 

   >**Nota**: Si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que utiliza en este laboratorio y seleccione **Crear almacenamiento**. 

1. En el panel de Cloud Shell, ejecute lo siguiente para iniciar sesión en el inquilino de Microsoft Entra:

   ```powershell
   Connect-AzureAD
   ```

1. En el panel de Cloud Shell, ejecute lo siguiente para recuperar el nombre de dominio DNS principal del inquilino de Microsoft Entra asociado a la suscripción de Azure:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. En el panel de Cloud Shell, ejecute lo siguiente para crear los usuarios de Microsoft Entra a los que se van a conceder privilegios elevados (reemplace el marcador de posición `<password>` por una contraseña aleatoria y compleja):

   > **Nota**: Asegúrese de recordar la contraseña que ha usado, ya que la necesitaremos más adelante en este y en laboratorios posteriores.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. En el panel de Cloud Shell, ejecute lo siguiente para asignar el rol Administrador global al primero de los usuarios de Microsoft Entra recién creados:

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **Nota**: En el módulo de PowerShell de Azure AD, el rol Administrador global se denomina Administrador de empresa.

1. En el panel de Cloud Shell, ejecute lo siguiente para conocer el nombre principal de usuario del usuario de Microsoft Entra recién creado:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **Nota**: Registre el nombre principal de usuario, ya que lo necesitará más adelante en este ejercicio. 

1. Cierre el panel de Cloud Shell.
1. En Azure Portal, busque y seleccione **Suscripciones** y, en la hoja **Suscripciones**, seleccione la suscripción de Azure que esté usando en este laboratorio. 
1. En la hoja que muestra las propiedades de la suscripción de Azure, seleccione **Control de acceso (IAM),****Agregar** y, por último, **Agregar asignación de roles**. 
1. En la hoja **Agregar asignación de roles**, seleccione **Propietario** y, a continuación, haga clic en **Siguiente**.
1. Haga clic en el hipervínculo **+Seleccionar miembros**.
1. En la hoja **Seleccionar miembros**, seleccione el elemento **aadadmin1**, haga clic en el botón **Seleccionar** y, a continuación, haga clic en **Siguiente**.
1. En la hoja **Revisar y asignar**, seleccione el botón **Revisar y asignar**.

   > **Nota**: Usaremos la cuenta **aadadmin1** más adelante en el laboratorio para administrar la suscripción de Azure y el inquilino de Microsoft Entra correspondiente desde una instancia de Microsoft Entra DS unida a una máquina virtual de Azure en Windows 10. 


#### Tarea 2: Implementar una instancia de Microsoft Entra DS mediante Azure Portal

1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Microsoft Entra Domain Services** y, en la hoja **Microsoft Entra Domain Services**, seleccione **+ Crear**. Se abrirá la hoja **Creación de Microsoft Entra Domain Services**.
1. En la pestaña **Datos básicos** de la hoja **Creación de Microsoft Entra Domain Services**, especifique la siguiente configuración y seleccione **Siguiente** (deje todo lo demás con los valores existentes):

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|Seleccione Crear nuevo > **az140-11a-RG**|
   |Nombre de dominio DNS|**adatum.com**|
   |Region|Nombre de la región donde desea hospedar la implementación de AVD|
   |SKU|**Estándar**|

   > **Nota**: Aunque técnicamente no es necesario, en general, debe asignar un nombre de dominio de Microsoft Entra DS diferente de cualquier espacio de nombres DNS local o de Azure existente.

1. En la pestaña **Redes** de la hoja **Creación de Microsoft Entra Domain Services**, junto a la lista desplegable **Red virtual**, seleccione **Crear nuevo**.
1. En la hoja **Crear red virtual**, asigne la siguiente configuración y seleccione **Aceptar**:

   |Configuración|Valor|
   |---|---|
   |Nombre|**az140-aadds-vnet11a**|
   |Intervalo de direcciones|**10.10.0.0/16**|
   |Nombre de subred|**aadds-Subnet**|
   |Intervalo de direcciones|**10.10.0.0/24**|

1. De nuevo en la pestaña **Redes** de la hoja **Crear red virtual**, seleccione **Siguiente** (deje todo lo demás con los valores existentes).
1. En la pestaña **Administración** de la hoja **Creación de Microsoft Entra Domain Services**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la pestaña **Sincronización** de la hoja **Creación de Microsoft Entra Domain Services**, asegúrese de que **Todo** está seleccionado y, a continuación, seleccione **Siguiente**.
1. En la pestaña **Configuración de seguridad** de la hoja **Creación de Microsoft Entra Domain Services**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la pestaña **Etiquetas** de la hoja **Creación de Microsoft Entra Domain Services**, acepte la configuración predeterminada y seleccione Siguiente
2. En la pestaña **Revisar y crear** de la hoja **Creación de Microsoft Entra Domain Services**, seleccione **Crear**. 
3. Revise la notificación relativa a la configuración, ya que no podrá cambiarla después de crear el dominio de Microsoft Entra, y seleccione **Aceptar**.

   >**Nota**: Los valores de configuración que no se podrán cambiar tras aprovisionar un dominio de Microsoft Entra DS son el nombre DNS, la suscripción de Azure, el grupo de recursos, la red virtual y la subred donde se hospedan los controladores de dominio, además del tipo de bosque.

   > **Nota**: Espere a que la implementación se complete antes de avanzar al siguiente ejercicio. Esto puede tardar unos 90 minutos. 

#### Tarea 3: Configurar la red y la identidad de la implementación de Microsoft Entra DS

1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Microsoft Entra Domain Services** y, en la hoja **Microsoft Entra Domain Services**, seleccione la entrada **adatum.com** para ir a la instancia de Microsoft Entra DS recién aprovisionada. 
1. En la hoja **adatum.com** de la instancia de Microsoft Entra DS, haga clic en la advertencia que indica que **se han detectado problemas de configuración en el dominio administrado**. 
1. En la hoja **adatum.com | Diagnósticos de configuración,** haga clic en **Ejecutar**.
1. En la sección **Validación**, expanda el panel **Registros DNS** y haga clic en **Corregir**.
1. En la hoja **Registros DNS**, haga clic en **Corregir** de nuevo.
1. Vuelva a la hoja **adatum.com** de la instancia de Microsoft Entra DS y, en la sección **Pasos de configuración necesarios**, revise la información relativa a la sincronización de hash de contraseña de Microsoft Entra DS. 

   > **Nota**: Todos los usuarios exclusivamente de nube existentes que necesiten poder acceder a los equipos de dominio de Microsoft Entra DS y a sus recursos deben cambiar sus contraseñas o hacer que se les restablezcan. Esto es pertinente también en la cuenta **aadadmin1** que creamos anteriormente en este laboratorio.

1. En el equipo de laboratorio, en Azure Portal, abra una sesión de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para averiguar el atributo objectID de la cuenta de usuario **aadadmin1** de Microsoft Entra:

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para restablecer la contraseña de la cuenta de usuario **aadadmin1**, cuyo objectID averiguamos en el paso anterior (reemplace el marcador de posición `<password>` por una contraseña aleatoria y compleja):

   > **Nota**: Asegúrese de recordar la contraseña que ha usado, ya que la necesitaremos más adelante en este y en laboratorios posteriores.

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **Nota**: En escenarios reales, normalmente estableceríamos el valor de **-ForceChangePasswordNextLogin** en $true. Hemos elegido **$false** en este caso para simplificar los pasos del laboratorio. 

1. Repita los dos pasos anteriores para restablecer la contraseña de la cuenta de usuario **wvdaadmin1**.


### Ejercicio 2: Configuración del entorno de dominio de Microsoft Entra DS
  
Las tareas principales de este ejercicio son las siguientes:

1. Implementar una máquina virtual de Azure que ejecuta Windows 10 mediante una plantilla de inicio rápido de Azure Resource Manager
1. Implementación de Azure Bastion
1. Revisión de la configuración predeterminada del dominio de Microsoft Entra DS
1. Creación de usuarios y grupos de AD DS que se sincronizarán con Microsoft Entra DS

#### Tarea 1: Implementar una máquina virtual de Azure que ejecuta Windows 10 mediante una plantilla de inicio rápido de Azure Resource Manager

1. En el equipo de laboratorio, en Azure Portal, en la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para agregar una subred denominada **cl-Subnet** a la red virtual **az140-aadds-vnet11a** que creamos en la tarea anterior:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. En el equipo de laboratorio, busque el archivo de parámetros **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** y ábralo con Visual Studio Code.

1.  En la línea 21, busque el valor del parámetro domainPassword. Actualice la contraseña existente en el archivo de parámetros para usar la contraseña que definimos anteriormente en este laboratorio para la cuenta de usuario **aadadmin1** y, a continuación, **guarde** el archivo.

1. En Azure Portal, en la barra de herramientas del panel de Cloud Shell, seleccione el icono **Cargar/Descargar archivos**; en el menú desplegable, seleccione **Cargar** y cargue los archivos **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** y **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** en el directorio principal de Cloud Shell.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para implementar una máquina virtual de Azure que ejecute Windows 10, que actuará como cliente de Azure Virtual Desktop, y unirla al dominio de Microsoft Entra DS:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **Nota**: La implementación puede tardar unos 10 minutos. Espere a que la implementación se complete antes de avanzar a la siguiente tarea. 


#### Tarea 2: Implementación de Azure Bastion 

> **Nota**: Azure Bastion permite la conexión a las máquinas virtuales de Azure sin puntos de conexión públicos que implementamos en la tarea anterior de este ejercicio, al tiempo que proporciona protección contra vulnerabilidades de seguridad de fuerza bruta que tienen como destino las credenciales de nivel de sistema operativo.

> **Nota**: Asegúrese de que el explorador tiene habilitada la función emergente.

1. En la ventana del explorador en que se muestra Azure Portal, abra otra pestaña y, en la pestaña del explorador, vaya a [**Azure Portal**](https://portal.azure.com).
1. En Azure Portal, seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda para abrir el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para agregar una subred denominada **AzureBastionSubnet** a la red virtual **az140-adds-vnet11** que creamos anteriormente en este ejercicio:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cierre el panel de Cloud Shell.
1. En Azure Portal, busque y seleccione **Bastions** y, en la hoja **Bastions**, seleccione **+ Crear**.
1. En la pestaña **Básico** de la hoja **Crear una instancia de Bastion**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|**az140-11a-RG**|
   |Nombre|**az140-11a-bastion**|
   |Region|La misma región de Azure donde implementamos los recursos en la tarea anterior de este ejercicio|
   |Nivel|**Basic**|
   |Red virtual|**az140-aadds-vnet11a**|
   |Subnet|**AzureBastionSubnet (10.10.254.0/24)**|
   |Dirección IP pública|**Crear nuevo**|
   |Nombre de la IP pública|**az140-aadds-vnet11a-ip**|

1. En la pestaña **Revisar y crear** de la hoja **Crear una instancia de Bastion**, seleccione **Crear**:

   > **Nota**: Espere a que la implementación se complete antes de avanzar a la siguiente tarea de este ejercicio. La implementación puede tardar unos 5 minutos.


#### Tarea 3: Revisión de la configuración predeterminada del dominio de Microsoft Entra DS

> **Nota**: Para poder iniciar sesión en el equipo recién unido a Microsoft Entra DS, debe agregar la cuenta de usuario con la que quiere iniciar sesión al grupo de Microsoft Entra de **administradores de controladores de dominio de AAD**. Este grupo de Microsoft Entra se crea automáticamente en el inquilino de Microsoft Entra asociado a la suscripción de Azure donde aprovisionamos la instancia de Microsoft Entra DS.

> **Nota**: Existe la posibilidad de rellenar este grupo con cuentas de usuario de Microsoft Entra existentes al aprovisionar una instancia de Microsoft Entra DS.

1. En el equipo de laboratorio, en Azure Portal, en el panel de Cloud Shell, ejecute lo siguiente para agregar la cuenta de usuario de Microsoft Entra **aadadmin1** al grupo de Microsoft Entra de **administradores de controladores de dominio de AAD**:

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. Cierre el panel de Cloud Shell.
1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11a**. Se abrirá la hoja **az140-cl-vm11a**.
1. En la hoja **az140-cl-vm11a**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion** y, en la pestaña **Bastion** de **az140-cl-vm11a**, proporcione las siguientes credenciales y seleccione **Conectar**:
1. Cuando se le solicite, inicie sesión como el usuario **aadadmin1** usando su nombre principal (que averiguamos anteriormente en este laboratorio) y la contraseña que definimos para esta cuenta de usuario al crearla anteriormente en el laboratorio.
1. En Bastion a la máquina virtual de Azure **az140-cl-vm11a**, inicie **Windows PowerShell ISE** como administrador y, en el panel de scripts **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar las Herramientas de administración remota del servidor relativas a Active Directory y DNS:

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **Nota**: Espere a que la instalación se complete antes de avanzar al siguiente paso. Esto puede tardar unos 2 minutos.

1. En Bastion a la máquina virtual de Azure **az140-cl-vm11a**, en el menú **Inicio**, vaya a la carpeta **Herramientas administrativas de Windows**, expándala y, en la lista de herramientas, inicie **Usuarios y equipos de Active Directory**. 
1. En la consola **Usuarios y equipos de Active Directory**, revise la jerarquía predeterminada, incluidas las unidades organizativas de **equipos de AADDC** y de **usuarios de AADDC**. Tenga en cuenta que la primera incluye la cuenta de equipo **az140-cl-vm11a** y la segunda, las cuentas de usuario sincronizadas desde el inquilino de Microsoft Entra asociado a la suscripción de Azure donde se hospeda la implementación de la instancia de Microsoft Entra DS. La unidad organizativa de **usuarios de AADDC** también incluye el grupo de **administradores de controladores de dominio de AAD** sincronizado desde el mismo inquilino de Microsoft Entra, junto con su pertenencia a grupos. Esta pertenencia no se puede modificar directamente en el dominio de Microsoft Entra DS, sino que, en su lugar, hay que administrarla en el inquilino de Microsoft Entra DS. Los cambios se sincronizan automáticamente con la réplica del grupo hospedado en el dominio de Microsoft Entra DS. 

   **Sugerencia:** Si en **Usuarios y equipos de Active Directory** no figura ningún contenido relacionado con el dominio, haga clic con el botón derecho en **Usuarios y equipos de Active Directory**, seleccione **Cambiar dominio** y elija el dominio **Adatum**.

   > **Nota**: Actualmente, el grupo solo incluye la cuenta de usuario **aadadmin1**.

1. En la consola **Usuarios y equipos de Active Directory**, en la unidad organizativa de **usuarios de AADDC**, seleccione la cuenta de usuario **aadadmin1**, abra su cuadro de diálogo **Propiedades**, cambie a la pestaña **Cuentas** y fíjese en que el sufijo de nombre principal de usuario coincide con el nombre de dominio DNS de Microsoft Entra principal, que no se puede modificar. 
1. En la consola **Usuarios y equipos de Active Directory**, revise el contenido de la unidad organizativa **Controladores de dominio** y fíjese en que incluye cuentas de equipo de dos controladores de dominio con nombres generados aleatoriamente. 

#### Tarea 4: Creación de usuarios y grupos de AD DS que se sincronizarán con Microsoft Entra DS

1. En Bastion a la máquina virtual de Azure **az140-cl-vm11a**, inicie Microsoft Edge, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión proporcionando el nombre principal de usuario de la cuenta de usuario **aadadmin1**, así como la contraseña que definimos anteriormente en este laboratorio como su contraseña.
1. En Azure Portal, abra **Cloud Shell**.
1. Cuando se le pida que seleccione **Bash** o **PowerShell**, seleccione **PowerShell**. 

   >**Nota**: Dado que esta es la primera vez que iniciamos **Cloud Shell** a través de la cuenta de usuario **aadadmin1**, hay configurar el directorio principal de Cloud Shell correspondiente. Si aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que estamos usando en este laboratorio y seleccione **Crear almacenamiento**. 

1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para iniciar sesión para autenticarse en el inquilino de Microsoft Entra:

   ```powershell
   Connect-AzureAD
   ```

1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para recuperar el nombre de dominio DNS principal del inquilino de Microsoft Entra asociado a la suscripción de Azure:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para crear las cuentas de usuario de Microsoft Entra que usaremos en los próximos laboratorios (reemplace el marcador de posición `<password>` por una contraseña aleatoria y compleja):

   > **Nota**: Asegúrese de recordar la contraseña que ha usado, ya que la necesitaremos más adelante en este y en laboratorios posteriores.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. En la sesión de PowerShell del panel Cloud Shell, ejecute lo siguiente para crear un grupo de Microsoft Entra denominado **az140-wvd-aadmins** y agregarlo a las cuentas de usuario **aadadmin1** y **wvdaadmin1**:

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. En el panel de Cloud Shell, repita el paso anterior para crear grupos de Microsoft Entra para los usuarios que usaremos en los próximos laboratorios y para agregarles cuentas de usuario de Microsoft Entra creadas previamente:

   >**Nota**: Nota: Debido al tamaño limitado del Portapapeles en la máquina virtual, no todos los cmdlets que aparecen se copiarán correctamente. Abra el Bloc de notas en la máquina virtual y copie en él todos los cmdlets usando la construcción Escribir texto, Escribir texto del Portapapeles perteneciente al control Rayo. Tras cerciorarse de que todos los cmdlets están en el Bloc de notas, córtelos y péguelos en bloques en Cloud Shell y ejecútelos.

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. Cierre el panel de Cloud Shell.
1. En Bastion a la máquina virtual de Azure **az140-cl-vm11a**, en la ventana de Microsoft Edge donde se muestra Azure Portal, busque y seleccione la hoja **Azure Active Directory**; en la hoja del inquilino de Microsoft Entra, en la barra de menús vertical de la izquierda, en la sección **Administrar**, seleccione **Usuarios** y, en la hoja **Usuarios \| Todos los usuarios**, compruebe que se han creado cuentas de usuario nuevas.
1. Vuelva a la hoja del inquilino de Microsoft Entra; en la barra de menús vertical de la izquierda, en la sección **Administrar**, seleccione **Grupos** y, en la hoja **Grupos \| Todos los grupos**, compruebe que se han creado cuentas de grupo nuevas.
1. En Bastion a la máquina virtual de Azure **az140-cl-vm11a**, cambie a la consola **Usuarios y equipos de Active Directory**; en la consola **Usuarios y equipos de Active Directory**, vaya a la unidad organizativa de **usuarios de AADDC** y compruebe que contiene las mismas cuentas de usuario y de grupo.

   >**Nota**: Es posible que tenga que actualizar la vista de la consola.
