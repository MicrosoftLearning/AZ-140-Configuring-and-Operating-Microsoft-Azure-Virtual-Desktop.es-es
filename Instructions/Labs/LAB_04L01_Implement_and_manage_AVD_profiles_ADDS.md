---
lab:
  title: "Laboratorio: Implementación y administración de perfiles de Azure\_Virtual\_Desktop (AD\_DS)"
  module: 'Module 4: Manage User Environments and Apps'
---

# <a name="lab---implement-and-manage-azure-virtual-desktop-profiles-ad-ds"></a>Laboratorio: Implementación y administración de perfiles de Azure Virtual Desktop (AD DS)
# <a name="student-lab-manual"></a>Manual de laboratorio para alumnos

## <a name="lab-dependencies"></a>Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Azure AD con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Azure AD asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**
- El laboratorio completado **Implementación y administración del almacenamiento para WVD (AD DS)**

## <a name="estimated-time"></a>Tiempo estimado

30 minutos

## <a name="lab-scenario"></a>Escenario del laboratorio

Debe implementar la administración de perfiles de Azure Virtual Desktop en un entorno de Active Directory Domain Services (AD DS).

## <a name="objectives"></a>Objetivos
  
Después de completar este laboratorio, podrá:

- Implementar perfiles basados en FSLogix para Azure Virtual Desktop

## <a name="lab-files"></a>Archivos de laboratorio

- None

## <a name="instructions"></a>Instructions

### <a name="exercise-1-implement-fslogix-based-profiles-for-azure-virtual-desktop"></a>Ejercicio 1: Implementar perfiles basados en FSLogix para Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Configuración de perfiles basados en FSLogix en máquinas virtuales del host de sesión de Azure Virtual Desktop
1. Prueba de perfiles basados en FSLogix con Azure Virtual Desktop
1. Eliminación de recursos de Azure implementados en el laboratorio

#### <a name="task-1-configure-fslogix-based-profiles-on-azure-virtual-desktop-session-host-vms"></a>Tarea 1: Configuración de perfiles basados en FSLogix en máquinas virtuales del host de sesión de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, haga clic en **az140-21-p1-0**.
1. En la hoja **az140-21-p1-0**, seleccione **Iniciar** y espere hasta que el estado de la máquina virtual cambie a **En ejecución**.
1. En la hoja **az140-21-p1-0**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-21-p1-0 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, inicie sesión con las siguientes credenciales:

   |Configuración|Value|
   |---|---|
   |Nombre de usuario|**student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Si se le solicita, inicie sesión con las credenciales de Azure AD de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. Dentro de la sesión de Escritorio remoto a **az140-21-p1-0**, en la ventana de Microsoft Edge que muestra Azure Portal, abra una sesión de PowerShell en el panel Cloud Shell. 
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para iniciar las máquinas virtuales de Azure del host de sesión de Azure Virtual Desktop que vamos a usar en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Nota**: Espere hasta que se ejecuten las máquinas virtuales de Azure antes de continuar con el paso siguiente.

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, inicie Microsoft Edge, vaya a la [página de descarga de FSLogix](https://aka.ms/fslogix_download), descargue los archivos binarios de instalación comprimida de FSLogix, extráigalos en la carpeta **C:\\Allfiles\\Labs\\04** (cree la carpeta si es necesario), vaya a la subcarpeta **x64\\Release**, haga doble clic en el archivo **FSLogixAppsSetup.exe** para iniciar el asistente de **Microsoft FSLogix Apps Setup** y avance paso a paso por la instalación de las aplicaciones de Microsoft FSLogix con la configuración predeterminada.

   > **Nota**: La instalación de FXLogic no es necesaria si la imagen ya la incluye.

3. En la sesión de Escritorio remoto a **az140-21-p1-0**, inicie **Windows PowerShell ISE** como administrador y, en el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar la última versión del módulo PowerShellGet (seleccione **Sí** cuando se le solicite confirmación):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Desde el panel del script de**Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar la última versión del módulo Az PowerShell (seleccione **Sí a todo** cuando se le solicite confirmación):

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. Desde la consola de**Administrador: Windows PowerShell ISE**, ejecute lo siguiente para modificar la directiva de ejecución:

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. Desde la consola de**Administrador: Windows PowerShell ISE**, ejecute lo siguiente para iniciar sesión en la suscripción de Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Cuando se le solicite, inicie sesión con las credenciales de Azure AD de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Escritorio remoto a **az140-21-p1-0**, desde el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para recuperar el nombre de la cuenta de Azure Storage que configuró anteriormente en este laboratorio:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, desde el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para configurar los parámetros de registro del perfil:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, haga clic con el botón derecho en **Inicio**, en el menú contextual, y seleccione **Ejecutar**; en el cuadro de diálogo **Ejecutar**, en el cuadro de texto **Abrir**, escriba lo siguiente y seleccione **Aceptar** para iniciar la consola **Usuarios y grupos locales**:

   ```cmd
   lusrmgr.msc
   ```

1. En la consola **Usuarios y grupos locales**, tenga en cuenta los cuatro grupos que comienzan con la cadena **FSLogix**:

   - Lista de exclusión ODFC de FSLogix
   - Lista de inclusión ODFC de FSLogix
   - Lista de exclusión del perfil de FSLogix
   - Lista de inclusión del perfil de FSLogix

1. En la consola **Usuarios y grupos locales**, en la lista de grupos, haga doble clic en el grupo **Lista de inclusión del perfil de FSLogix**, compruebe que incluye el grupo **\\Todos**, y seleccione **Aceptar** para cerrar la ventana del grupo **Propiedades**. 
1. En la consola **Usuarios y grupos locales**, en la lista de grupos, haga doble clic en el grupo **Lista de exclusión del perfil de FSLogix**, compruebe que no incluye miembros del grupo de manera predeterminada y seleccione **Aceptar** para cerrar la ventana del grupo **Propiedades**. 

   > **Nota**: Para proporcionar una experiencia de usuario coherente, debe instalar y configurar componentes de FSLogix en todos los hosts de sesión de Azure Virtual Desktop. Realizará esta tarea de forma desatendida en los demás hosts de sesión de nuestro entorno de laboratorio. 

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, desde el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar los componentes de FSLogix en los hosts de sesión **az140-21-p1-1** y **az140-21-p1-2**:

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **Nota**: Espere a que termine la ejecución del script. Esto puede tardar unos 2 minutos.

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, desde el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para configurar los ajustes de registro del perfil en los hosts de sesión **az140-21-p1-1** y **az140-21-p1-1**:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **Nota**: Antes de probar la funcionalidad del perfil basado en FSLogix, debe quitar el perfil almacenado en caché local de la cuenta **aduser1\\ de ADATUM** que va a usar para realizar pruebas desde los hosts de sesión de Azure Virtual Desktop que usó en el laboratorio anterior.

1. En la sesión de Escritorio remoto a **az140-21-p1-0**, desde el panel del script de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para eliminar el perfil almacenado localmente en caché de la cuenta **aduser1\\de ADATUM** en todas las máquinas virtuales de Azure que sirven como hosts de sesión:

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### <a name="task-2-test-fslogix-based-profiles-with-azure-virtual-desktop"></a>Tarea 2: Prueba de perfiles basados en FSLogix con Azure Virtual Desktop

1. Cambie al equipo de laboratorio y desde él, en la ventana del explorador donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11**.
1. En la hoja **az140-cl-vm11**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-cl-vm11 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Value|
   |---|---|
   |Nombre de usuario|**Student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-cl-vm11**, haga clic en **Inicio** y, en el menú **Inicio**,haga clic en la aplicación del cliente **Escritorio remoto**.
1. En la sesión de Escritorio remoto a **az140-cl-vm11**, en la ventana del cliente **Escritorio remoto**, seleccione **Suscribirse** y, cuando se le solicite, inicie sesión con las credenciales **aduser1**.

 >**Nota**  Si no se le pide que se suscriba, es posible que tenga que cancelar la suscripción anterior.
3. en la lista de aplicaciones, haga doble clic en **Símbolo del sistema**, y, cuando se le solicite, proporcione la contraseña de la cuenta **aduser1** y compruebe que se abra correctamente una ventana del **Símbolo del sistema**.
4. En la esquina superior izquierda de la ventana del **Símbolo del sistema**, haga clic con el botón derecho en el icono del **Símbolo del sistema** y, en el menú desplegable, seleccione **Propiedades**.
5. En el cuadro de diálogo **Propiedades del símbolo del sistema**, seleccione la pestaña **Fuente**, modifique la configuración de tamaño y fuente y seleccione **Aceptar**.
6. En la ventana **Símbolo del sistema**, escriba **logoff** y presione la tecla **Intro** para cerrar sesión desde la sesión de Escritorio remoto.
7. Dentro de la sesión de Escritorio remoto a **az140-cl-vm11**, en la ventana del cliente **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **SessionDesktop** en az140-21-ws1 y compruebe que inicia una sesión de Escritorio remoto. 
8. En la sesión **SessionDesktop**, haga clic con el botón derecho en **Inicio**, y, en el menú contextual, seleccione **Ejecutar**; en el cuadro de diálogo **Ejecutar**, en el cuadro de texto **Abrir**, escriba **cmd** y seleccione **Aceptar** para iniciar una ventana del **Símbolo del sistema**:
9. Compruebe que la configuración de la ventana **Símbolo del sistema** coincida con las que configuró anteriormente en esta tarea.
10. En la sesión **SessionDesktop**, minimice todas las ventanas, haga clic con el botón derecho en el escritorio, y, en el menú contextual, seleccione **Nuevo** y, en el menú en cascada, seleccione **Acceso directo**. 
11. En la página **¿A qué elemento le desea crear un acceso directo?** del Asistente **Crear acceso directo**, en el cuadro de texto **Type the location of the item** (Escribir la ubicación del elemento), escriba **Bloc de notas** y seleccione **Siguiente**.
12. En la página **What would you like to name the shortcut** (¿Qué nombre quiere asignar al acceso directo?) del Asistente **Crear acceso directo**, en el cuadro de texto **Type a name for this shortcut** (Escribir el nombre del acceso directo), escriba **Bloc de notas** y seleccione **Finalizar**.
13. En la sesión **SessionDesktop**, haga clic con el botón derecho en **Inicio**, en el menú contextual, seleccione **Apagar o cerrar sesión** y, a continuación, en el menú en cascada, seleccione **Cerrar sesión**.
14. De nuevo en la sesión de Escritorio remoto a **az140-cl-vm11**, en la ventana del cliente **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **SessionDesktop** para iniciar una nueva sesión de Escritorio remoto. 
15. En la sesión **SessionDesktop**, compruebe que el acceso directo del **Bloc de notas** aparece en el escritorio.
16. En la sesión **SessionDesktop**, haga clic con el botón derecho en **Inicio**, en el menú contextual, seleccione **Apagar o cerrar sesión** y, a continuación, en el menú en cascada, seleccione **Cerrar sesión**.
17. Cambie al equipo de laboratorio y, en la ventana de Microsoft Edge que muestra el Azure Portal, vaya a la hoja **Cuentas de almacenamiento** y seleccione la entrada que representa la cuenta de almacenamiento que creó en el ejercicio anterior.
18. En la hoja Cuenta de almacenamiento, en la sección **Servicios de archivos**, seleccione **Recursos compartidos de archivos** y, a continuación, en la lista de recursos compartidos de archivos, seleccione **az140-22-profiles**. 
19. En la hoja **az140-22-profiles**, compruebe que su contenido incluye una carpeta cuyo nombre consta de una combinación del identificador de seguridad (SID) de la cuenta **aduser1\\ de ADATUM** seguida del sufijo **_aduser1**.
20. Seleccione la carpeta que identificó en el paso anterior y tenga en cuenta que contiene un único archivo denominado **Profile_aduser1.vhd**.

### <a name="exercise-2-stop-and-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>Ejercicio 2: Detener y desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detener y desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignaremos las máquinas virtuales de Azure aprovisionadas en este laboratorio para reducir al mínimo los cargos de proceso correspondientes.

#### <a name="task-1-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>Tarea 1: Desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel de Cloud Shell, ejecute lo siguiente para mostrar todas las máquinas virtuales de Azure que hemos creado y usado en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Desde la sesión de PowerShell, en el panel de Cloud Shell, ejecute lo siguiente para detener y desasignar todas las máquinas virtuales de Azure que hemos creado y usado en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.
