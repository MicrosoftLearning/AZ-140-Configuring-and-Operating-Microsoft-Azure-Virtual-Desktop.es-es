---
lab:
  title: "Laboratorio: Implementación y administración de almacenamiento para AVD (AD\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratorio: Implementación y administración de almacenamiento para AVD (AD DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Microsoft Entra asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**

## Tiempo estimado

30 minutos

## Escenario del laboratorio

Necesita implementar y administrar el almacenamiento para una implementación de Azure Virtual Desktop en un entorno AD DS.

## Objetivos
  
Después de completar este laboratorio, podrá:

- Configurar Azure Files para almacenar contenedores de perfiles de Azure Virtual Desktop

## Archivos de laboratorio

- None

## Instrucciones

### Ejercicio 1: Configurar Azure Files para almacenar contenedores de perfiles de Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Creación de una cuenta de Azure Storage
1. Crear un recurso compartido de Azure Files
1. Habilitar la autenticación de AD DS para la cuenta de Azure Storage 
1. Configurar los permisos basados en RBAC de Azure Files
1. Configurar los permisos del sistema de archivos en Azure Files

#### Tarea 1: Crear una cuenta de Azure Storage.

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, haga clic en **az140-dc-vm11**.
1. En el panel **az140-dc-vm11**, seleccione **Conectar**, en el menú desplegable, seleccione **Conectar via Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Bastion a **az140-dc-vm11**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Si se le solicita, inicie sesión con las credenciales de Microsoft Entra de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. Dentro de la sesión de Bastion a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra el portal de Azure, busque y seleccione **Cuentas de almacenamiento** y, en el panel **Cuentas de almacenamiento**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear cuenta de almacenamiento**, configure las siguientes opciones (deje las demás con los valores predeterminados):

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|crear un **nuevo** grupo de recursos denominado **az140-22-RG**|
   |Nombre de la cuenta de almacenamiento|cualquier nombre único global de entre 3 y 15 caracteres, que conste de letras en minúsculas y dígitos, y que empiece por una letra.|
   |Region|nombre de una región de Azure que hospeda el entorno de laboratorio de Azure Virtual Desktop.|
   |Rendimiento|**Estándar**|
   |Redundancia|**Almacenamiento con redundancia geográfica (GRS)**|
   |Habilite el acceso de lectura a los datos en el caso de que la región no esté disponible.|enabled|

   >**Nota**: Asegúrese de que el nombre de cuenta de almacenamiento no supere los 15 caracteres. El nombre se usará para crear una cuenta de equipo en el dominio de Active Directory Domain Services (AD DS) integrado con el inquilino de Microsoft Entra asociado a la suscripción de Azure que incluye la cuenta de almacenamiento. Esto permitirá la autenticación basada en AD DS al acceder a los recursos compartidos de archivos hospedados en esta cuenta de almacenamiento.

1. En la pestaña **Aspectos básicos** de la hoja **Crear cuenta de almacenamiento**, seleccione **Revisar y crear**, espere a que se complete el proceso de validación y, luego, seleccione **Crear**.

   >**Nota**: Espere a que se cree la cuenta de almacenamiento. Este proceso tardará alrededor de 2 minutos.

#### Tarea 2: Crear un recurso compartido de Azure Files

1. En la sesión de Bastion a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra el portal de Azure, navegue de nuevo a el panel **Cuentas de almacenamiento** y seleccione la entrada que representa la cuenta de almacenamiento recién creada.
1. En la hoja Cuenta de almacenamiento, en la sección **Almacenamiento de datos**, seleccione **Recurso compartido de archivos** y, después, seleccione **+ Recurso compartido de archivos**.
1. En el panel **Nuevo recurso compartido de archivos**, especifique la siguiente configuración y seleccione **Siguiente : Copia de seguridad > **(deje los demás ajustes con sus valores por defecto):

   |Configuración|Value|
   |---|---|
   |Nombre|**az140-22-profiles**|
   |Nivel de acceso|**Transacción optimizada**|

1. En el panel **Copia de seguridad**, anule la selección de la casilla **Habilitar copia de seguridad**, seleccione **Revisar + Crear**, espere a que finalice el proceso de validación, y luego seleccione **Crear**.

#### Tarea 3: Habilitar la autenticación de AD DS para la cuenta de Azure Storage 

1. Dentro de la sesión de Bastion a **az140-dc-vm11**, abra otra pestaña en la ventana de Microsoft Edge, vaya al [repositorio GitHub de ejemplos de Azure Files](https://github.com/Azure-Samples/azure-files-samples/releases), descargue [la versión más reciente del módulo PowerShell comprimido**AzFilesHybrid.zip**y extraiga su contenido en la carpeta **C:\\ Todos los archivos\\Laboratorio\\02**(cree la carpeta si es necesario).
1. En la sesión de Bastion a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador y, en la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para quitar el flujo de datos alternativo **Zone.Identifier**, que tiene un valor de **3** para indicar que se descargó de Internet:

   ```powershell
   Get-ChildItem -Path C:\Allfiles\Labs\02 -File -Recurse | Unblock-File
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para establecer las variables necesarias para ejecutar el script siguiente:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear un objeto de equipo de AD DS que represente la cuenta de Azure Storage que creó recientemente en esta tarea y se utilice para implementar su autenticación de AD DS:

   >**Nota**: Si recibe un error al ejecutar este bloque de script, asegúrese de que se encuentra en el mismo directorio que el script CopyToPSPath.ps1. En función de cómo se extrajeron los archivos anteriormente en este laboratorio, podrían estar en una subcarpeta denominada AzFilesHybrid. En el contexto de PowerShell, cambie los directorios a la carpeta mediante **cd AzFilesHybrid**.

   ```powershell
   Set-Location -Path 'C:\Allfiles\Labs\02'
   .\CopyToPSPath.ps1 
   Import-Module -Name AzFilesHybrid
   Join-AzStorageAccountForAuth `
      -ResourceGroupName $ResourceGroupName `
      -StorageAccountName $StorageAccountName `
      -DomainAccountType 'ComputerAccount' `
      -OrganizationalUnitDistinguishedName 'OU=WVDInfra,DC=adatum,DC=com'
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para verificar que la autenticación de AD DS está habilitada en la cuenta de Azure Storage:

   ```powershell
   $storageaccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
   $storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
   $storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions
   ```

1. Compruebe que la salida del comando `$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties` devuelve `AD`, que representa el servicio del directorio de la cuenta de almacenamiento y que la salida del comando `$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions`, que representa la información del dominio del directorio, es similar al siguiente formato (los valores de `DomainGuid`, `DomainSid` y `AzureStorageSid` variarán):

   ```
   DomainName        : adatum.com
   NetBiosDomainName : adatum.com
   ForestName        : adatum.com
   DomainGuid        : 47c93969-9b12-4e01-ab81-1508cae3ddc8
   DomainSid         : S-1-5-21-1102940778-2483248400-1820931179
   AzureStorageSid   : S-1-5-21-1102940778-2483248400-1820931179-2109
   ```

1. Dentro de la sesión de Bastion a **az140-dc-vm11**, cambie a la ventana de Microsoft Edge que muestra el portal de Azure, en el panel que muestra la cuenta de almacenamiento, seleccione **Archivos compartidos** y verifique que el ajuste **Acceso basado en identidad** esté **Configurado**.

   >**Nota**: Es posible que tenga que actualizar la página del explorador para que el cambio se refleje dentro de Azure Portal.

#### Tarea 4: Configurar los permisos basados en RBAC de Azure Files

1. Dentro de la sesión de Bastion a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra el Azure portal, en el panel que muestra las propiedades de la cuenta de almacenamiento que creó anteriormente en este ejercicio, en el menú vertical del lado izquierdo, en la sección **Almacenamiento de datos**, seleccione **Archivos compartidos**.
1. En la hoja **Recursos compartidos de archivos**, en la lista de recursos compartidos, seleccione la entrada **az140-22-profiles**.
1. En la hoja **az140-22-profiles**, en el menú vertical del lado izquierdo, seleccione **Control de acceso (IAM)**.
1. En la hoja **Control de acceso (IAM)** de la cuenta de almacenamiento, seleccione **+ Agregar** y, en el menú desplegable, seleccione **Agregar asignación de roles**. 
1. En el panel **Agregar asignación de roles** en la pestaña **Rol**, especifique la siguiente configuración y seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Rol de función de trabajo|**Colaborador de recursos compartidos de SMB de datos de archivos de Storage**|

1. En el panel **Agregar asignación de roles**, en la pestaña **Miembros**, haga clic en **+ Seleccionar miembros**, especifique la siguiente configuración y haga clic en **Seleccionar**. 

   |Configuración|Valor|
   |---|---|
   |Seleccionar|**az140-wvd-users**|
1. En el panel **Agregar asignación de roles**, seleccione **Revisar + asignar**, y luego, seleccione **Revisar + asignar**.
1. En la hoja **Control de acceso (IAM)** de la cuenta de almacenamiento, seleccione **+ Agregar** y, en el menú desplegable, seleccione **Agregar asignación de roles**. 
1. En el panel **Agregar asignación de roles** en la pestaña **Rol**, especifique la siguiente configuración y seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Rol de función de trabajo|**Colaborador elevado de recursos compartidos de SMB de datos de archivos de Storage**|

1. En el panel **Agregar asignación de roles**, en la pestaña **Miembros**, haga clic en **+ Seleccionar miembros**, especifique la siguiente configuración y haga clic en **Seleccionar**. 

   |Configuración|Valor|
   |---|---|
   |Seleccionar|**az140-wvd-admins**|
1. En el panel **Agregar asignación de roles**, seleccione **Revisar + asignar**, y luego, seleccione **Revisar + asignar**.

#### Tarea 5: Configurar los permisos del sistema de archivos en Azure Files

1. En la sesión de Bastion a **az140-dc-vm11**, cambie a la ventana **Administrador: Windows PowerShell ISE** y, en la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear una variable que haga referencia al nombre y la clave de la cuenta de almacenamiento que creó anteriormente en este ejercicio:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0]
   $storageAccountName = $storageAccount.StorageAccountName
   $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear una asignación de unidad al recurso compartido de archivos que creó anteriormente en este ejercicio:

   ```powershell
   $fileShareName = 'az140-22-profiles'
   net use Z: "\\$storageAccountName.file.core.windows.net\$fileShareName" /u:AZURE\$storageAccountName $storageAccountKey
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para visualizar los permisos de sistema del archivo actual:

   ```powershell
   icacls Z:
   ```

   >**Nota**: De forma predeterminada, tanto los **Usuarios autenticados de \\NT Authority** y los **Usuarios de \\BUILTIN** tendrían permisos que permiten a los usuarios leer los contenedores de perfil de otros usuarios. En su lugar, los quitará y agregará los permisos mínimos necesarios.

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para ajustar los permisos del sistema de archivos y que cumplan con el principio de menor privilegio:

   ```powershell
   $permissions = 'ADATUM\az140-wvd-admins'+':(F)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'ADATUM\az140-wvd-users'+':(M)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'Creator Owner'+':(OI)(CI)(IO)(M)'
   cmd /c icacls Z: /grant $permissions
   icacls Z: /remove 'Authenticated Users'
   icacls Z: /remove 'Builtin\Users'
   ```

   >**Nota**: Como alternativa, puede establecer permisos mediante el Explorador de archivos.
