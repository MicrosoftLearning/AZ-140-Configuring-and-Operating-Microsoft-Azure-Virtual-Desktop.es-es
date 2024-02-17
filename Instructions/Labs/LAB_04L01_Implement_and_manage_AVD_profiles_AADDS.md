---
lab:
  title: "Laboratorio: implementación y administración de perfiles de Azure\_Virtual\_Desktop (Microsoft Entra DS)"
  module: 'Module 4: Manage User Environments and Apps'
---

# Laboratorio: implementación y administración de perfiles de Azure Virtual Desktop (Microsoft Entra DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure
- Una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure y con el rol Propietario o Colaborador en la suscripción de Azure
- Un entorno de Azure Virtual Desktop aprovisionado en el laboratorio **Introducción a Azure Virtual Desktop (Microsoft Entra DS)**

## Tiempo estimado

30 minutos

## Escenario del laboratorio

Debe implementar la administración de perfiles de Azure Virtual Desktop en un entorno de Microsoft Entra DS.

## Objetivos
  
Después de completar este laboratorio, podrá:

- Configurar Azure Files para almacenar contenedores de perfiles de Azure Virtual Desktop en un entorno de Microsoft Entra DS
- Implementar perfiles basados en FSLogix para Azure Virtual Desktop en un entorno de Microsoft Entra DS

## Archivos de laboratorio

- None

## Instrucciones

### Ejercicio 1: Implementar perfiles basados en FSLogix para Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Configurar el grupo administradores locales en máquinas virtuales del host de sesión de Azure Virtual Desktop
1. Configuración de perfiles basados en FSLogix en máquinas virtuales del host de sesión de Azure Virtual Desktop
1. Probar perfiles basados en FSLogix con Azure Virtual Desktop
1. Eliminar recursos de laboratorio de Azure

#### Tarea 1: Configurar el grupo administradores locales en máquinas virtuales del host de sesión de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. Seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda en Azure Portal para abrir el panel de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **PowerShell**. 

   >**Nota**: Si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que utiliza en este laboratorio y seleccione **Crear almacenamiento**. 

1. En la sesión de PowerShell del panel de **Cloud Shell**, ejecute lo siguiente para iniciar las máquinas virtuales de Azure del host de sesión de Azure Virtual Desktop que vamos a usar en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Start-AzVM
   ```

   >**Nota**: Espere hasta que se ejecuten las máquinas virtuales de Azure antes de continuar con el paso siguiente.
   
      
1. Desde la sesión de PowerShell en el panel de **Cloud Shell**, ejecute lo siguiente para habilitar PowerShell Remoting en los hosts de sesión.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Enable-AzVMPSRemoting
   ```
   
1. Cierre de Cloud Shell
1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11a**. Se abrirá la hoja **az140-cl-vm11a**.
1. En la hoja **az140-cl-vm11a**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-cl-vm11a \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**aadadmin1@adatum.com**|
   |Contraseña|Contraseña configurada anteriormente|

1. En la sesión de Bastion a **az140-cl-vm11a**, en el menú Inicio, vaya a la carpeta **Herramientas de administración de Windows**, expándala y seleccione **Usuarios y equipos de Active Directory**.
1. En la consola de **Usuarios y equipos de Active Directory**, haga clic con el botón derecho en el nodo de dominio, seleccione **Nuevo**, seguido de **Unidad organizativa**; en el cuadro de diálogo **New Object - Organizational Unit** (Nuevo objeto: Unidad organizativa), en el cuadro de texto **Nombre**, escriba **Usuarios de ADDC** y seleccione **Aceptar**.
1. En la consola de **Usuarios y equipos de Active Directory**, haga clic con el botón derecho en **ADDC Users** (Usuarios de ADDC), seleccione **Nuevo** seguido de **Grupo**; en el cuadro de diálogo **New Object - Group** (Nuevo objeto: Grupo), especifique la configuración siguiente y seleccione **Aceptar**:

   |Configuración|Valor|
   |---|---|
   |Nombre del grupo|**Administradores locales**|
   |Nombre del grupo (previo a Windows 2000)|**Administradores locales**|
   |Ámbito de grupo|**Operaciones**|
   |Tipo de grupo|**Seguridad**|

1. En la consola de **Usuarios y equipos de Active Directory**, muestre las propiedades del grupo **Administradores locales**, cambie a la pestaña **Miembros**, seleccione **Agregar**; en el cuadro de diálogo **Select Users, Contacts, Computers, Service Accounts, or Groups** (Seleccionar usuarios, contactos, equipos, cuentas de servicio o grupos), en el cuadro de diálogo **Enter the object names to select** (Escribir los nombres de objeto que desea seleccionar), escriba **aadadmin1;wvdaadmin1** y seleccione **Aceptar**.
1. En la sesión de Bastion a **az140-cl-vm11a**, en el menú Inicio, vaya a la carpeta **Herramientas de administración de Windows**, expándala y seleccione **Administración de la directiva de grupo**.
1. En la consola de **Administración de directivas de grupo**, vaya a la unidad organizativa **Equipos de AADDC**, haga clic con el botón derecho en el icono **GPO de equipos de AADDC** y seleccione **Editar**.
1. En la consola de **Editor de administración de directivas de grupo**, expanda **Configuración del equipo**, **Directivas**, **Configuración de Windows**, **Configuración de seguridad**, haga clic con el botón derecho en **Grupos restringidos** y seleccione **Agregar grupo**.
1. En el cuadro de diálogo **Agregar grupo**, en el cuadro de texto **Grupo**, seleccione **Examinar**; en el cuadro de diálogo **Seleccionar grupos**, en la opción **Enter the object names to select** (Escribir los nombres de objeto que desea seleccionar), escriba **Administradores locales** y seleccione **Aceptar**.
1. De nuevo en el cuadro de diálogo **Agregar grupo**, seleccione **Aceptar**.
1. En el cuadro de diálogo **ADATUM\Propiedades de administradores locales**, en la sección etiquetada **This group is a member of** (Este grupo es miembro de), seleccione **Agregar**; en el cuadro de diálogo **Pertenencia a grupos**, escriba **Administradores**, seleccione **Aceptar** y luego **Aceptar** de nuevo para finalizar el cambio.

   >**Nota**: Asegúrese de usar la sección etiquetada **This group is a member of** (Este grupo es miembro de).

1. En la sesión de Bastion a la máquina virtual de Azure az140-cl-vm11a, inicie PowerShell ISE como administrador y ejecute lo siguiente para reiniciar los dos hosts de Azure Virtual Desktop a fin de desencadenar el procesamiento de la directiva de grupo:

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. Espere a que se complete el script. Este proceso tardará aproximadamente 3 minutos.

#### Tarea 2: Configuración de perfiles basados en FSLogix en máquinas virtuales del host de sesión de Azure Virtual Desktop

1. En la sesión de Bastion a **az140-cl-vm11a**, inicie una sesión de Escritorio remoto en **az140-21-p1-0** y, cuando se le solicite, inicie sesión con el nombre de usuario **ADATUM\wvdaadmin1** y la contraseña que estableció al crear esta cuenta de usuario. 

   > **Nota**: Si la conexión RDP no se puede conectar, use Azure Portal para conectarse a la máquina virtual mediante Bastion.

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, inicie Microsoft Edge, vaya a la [página de descarga de FSLogix](https://aka.ms/fslogix_download), descargue los archivos binarios de instalación comprimidos de FSLogix, extráigalos en la carpeta **C:\\Source**, vaya a la subcarpeta **x64\\Release** y use **FSLogixAppsSetup.exe** para instalar aplicaciones de Microsoft FSLogix con la configuración predeterminada.

   > **Nota**: Es posible que la instalación de FXLogic no sea necesaria, en función de si la imagen ya la incluye. La instalación de FXLogic requiere que se reinicie el equipo.

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, inicie **Windows PowerShell ISE** como administrador y, en el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar la última versión del módulo PowerShellGet (seleccione **Sí** cuando se le solicite confirmación):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar la última versión del módulo Az PowerShell (seleccione **Sí a todo** cuando se le solicite confirmación):

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para modificar la directiva de ejecución:

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para iniciar sesión en la suscripción de Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Cuando se le solicite, inicie sesión con las credenciales de Microsoft Entra de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para recuperar el nombre de la cuenta de Azure Storage que configuró anteriormente en este laboratorio:

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, desde el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para configurar los parámetros de registro del perfil:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```
   >**Nota**: Si el comando genera un error, continúe con el paso siguiente.
   
1. En la sesión de Escritorio remoto a **az140-21-p1-0**, haga clic con el botón derecho en **Inicio**; en el menú contextual, seleccione **Ejecutar**; en el cuadro de diálogo **Ejecutar**, en el cuadro de texto **Abrir**, escriba lo siguiente y seleccione **Aceptar** para iniciar la ventana **Usuarios y grupos locales**:

   ```cmd
   lusrmgr.msc
   ```

1. En la consola **Usuarios y grupos locales**, tenga en cuenta los cuatro grupos que comienzan con la cadena **FSLogix**:

   - Lista de exclusión ODFC de FSLogix
   - Lista de inclusión ODFC de FSLogix
   - Lista de exclusión del perfil de FSLogix
   - Lista de inclusión del perfil de FSLogix

1. En la consola de **Usuarios y grupos locales**, haga doble clic en la entrada del grupo **FSLogix Profile Include List** (Lista de inclusión del perfil de FSLogix ), tenga en cuenta que incluye el grupo **\\Todos** y seleccione **Aceptar** para cerrar la ventana **Propiedades** del grupo. 
1. En la consola de **Usuarios y grupos locales**, haga doble clic en la entrada del grupo **FSLogix Profile Exclude List** (Lista de exclusión del perfil de FSLogix); tenga en cuenta que no incluye miembros del grupo de manera predeterminada y seleccione **Aceptar** para cerrar la ventana **Propiedades** del grupo. 

   > **Nota**: Para proporcionar una experiencia de usuario coherente, debe instalar y configurar componentes de FSLogix en todos los hosts de sesión de Azure Virtual Desktop. Realizará esta tarea de forma desatendida en el otro host de sesión de nuestro entorno de laboratorio. 

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, desde el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar los componentes de FSLogix en el host de sesión **az140-21-p1-1**:

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, inicie **Windows PowerShell ISE** como administrador y, en el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para configurar los ajustes de registro del perfil en el host de sesión **az140-21-p1-1**:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **Nota**: Antes de probar la función del perfil basado en FSLogix, debe quitar el perfil almacenado en caché local de la cuenta ADATUM\wvdaadmin1 que va a usar para realizar pruebas desde los hosts de sesión de Azure Virtual Desktop que usó en el laboratorio anterior.

1. Cambie a la sesión de Bastion a **az140-cl-vm11a**, en la sesión de Bastion a **az140-cl-vm11a**, cambie a la ventana **Administrador: Windows PowerShell ISE** y, en la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para eliminar el perfil almacenado localmente en caché de la cuenta ADATUM\aaduser1:

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### Tarea 3: Prueba de perfiles basados en FSLogix con Azure Virtual Desktop

1. En la sesión de Bastion a **az140-cl-vm11a**, cambie al cliente de Escritorio remoto.
1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de cliente de **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **Símbolo del sistema**; cuando se le solicite, proporcione la contraseña y compruebe que inicia una ventana de **Símbolo del sistema**. 

   > **Nota**: Inicialmente, la aplicación puede tardar unos minutos en iniciarse pero, posteriormente, el inicio de la aplicación debe ser mucho más rápido.

1. En la esquina superior izquierda de la ventana **Símbolo del sistema**, haga clic con el botón derecho en el icono **Símbolo del sistema** y, en el menú desplegable, seleccione **Propiedades**.
1. En el cuadro de diálogo **Propiedades del símbolo del sistema**, seleccione la pestaña **Fuente**, modifique la configuración de tamaño y fuente y seleccione **Aceptar**.
1. En la ventana **Símbolo del sistema**, escriba **logoff** y presione la tecla **Intro** para cerrar la sesión desde la sesión de Escritorio remoto.
1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana del cliente **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **SessionDesktop** y compruebe que inicia sesión en Escritorio remoto. 
1. En la sesión **SessionDesktop**, haga clic con el botón derecho en **Inicio**, y, en el menú contextual, seleccione **Ejecutar**; en el cuadro de diálogo **Ejecutar**, en el cuadro de texto **Abrir**, escriba **cmd** y seleccione **Aceptar** para iniciar una ventana del **Símbolo del sistema**:
1. Compruebe que las propiedades de la ventana **Símbolo del sistema** coinciden con las que estableció anteriormente en esta tarea.
1. En la sesión **SessionDesktop**, minimice todas las ventanas, haga clic con el botón derecho en el escritorio, y, en el menú contextual, seleccione **Nuevo** y, en el menú en cascada, seleccione **Acceso directo**. 
1. En la página **¿A qué elemento le desea crear un acceso directo?** del Asistente **Crear acceso directo**, en el cuadro de texto **Type the location of the item** (Escribir la ubicación del elemento), escriba **Bloc de notas** y seleccione **Siguiente**.
1. En la página **What would you like to name the shortcut** (¿Qué nombre quiere asignar al acceso directo?) del Asistente **Crear acceso directo**, en el cuadro de texto **Type a name for this shortcut** (Escribir el nombre del acceso directo), escriba **Bloc de notas** y seleccione **Finalizar**.
1. En la sesión **SessionDesktop**, haga clic con el botón derecho en **Inicio**, en el menú contextual, seleccione **Apagar o cerrar sesión** y, a continuación, en el menú en cascada, seleccione **Cerrar sesión**.
1. De nuevo en la sesión de Bastion a **az140-cl-vm11a**, en la ventana del cliente **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **SessionDesktop** para iniciar una nueva sesión de Escritorio remoto. 
1. En la sesión **SessionDesktop**, compruebe que el acceso directo del **Bloc de notas** aparece en el escritorio.
1. En la sesión **SessionDesktop**, haga clic con el botón derecho en **Inicio**, en el menú contextual, seleccione **Apagar o cerrar sesión** y, a continuación, en el menú en cascada, seleccione **Cerrar sesión**.
1. Cambie a la sesión de Bastion a **az140-cl-vm11a** y cambie también a la ventana de Microsoft Edge en la que se muestra Azure Portal.
1. En la ventana de Microsoft Edge en la que se muestra Azure Portal, vaya a la hoja **Cuentas de almacenamiento** y seleccione la entrada que representa la cuenta de almacenamiento que creó en el ejercicio anterior.
1. En la hoja Cuenta de almacenamiento, en la sección **Servicios de archivo**, seleccione **Recursos compartidos de archivos** y, después, en la lista de recursos compartidos de archivos, seleccione **az140-22-profiles**. 
1. En la hoja **az140-22-profiles**, seleccione **Examinar** y compruebe que su contenido incluye una carpeta cuyo nombre consiste en una combinación del Identificador de seguridad (SID) de la cuenta **ADATUM\\aaduser1** seguido del sufijo **_aaduser1**.
1. Seleccione la carpeta que identificó en el paso anterior y tenga en cuenta que contiene un único archivo denominado **Profile_aaduser1.vhd**.

### Ejercicio 2: Eliminar recursos de laboratorio de Azure (opcional)

1. Quite la implementación de Microsoft Entra DS; para ello, siga las instrucciones descritas en [Eliminación de un dominio administrado de Azure Active Directory Domain Services mediante Azure Portal]( https://docs.microsoft.com/en-us/azure/active-directory-domain-services/delete-aadds).
1. Quite todos los grupos de recursos de Azure que ha aprovisionado en los laboratorios de Microsoft Entra DS de este curso; para ello, siga las instrucciones descritas en [Eliminación de grupos de recursos y recursos en Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal).
