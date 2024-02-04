---
lab:
  title: "Laboratorio: Empaquetado de aplicaciones de Azure Virtual Desktop (AD\_DS)"
  module: 'Module 4: Manage User Environments and Apps'
---

# Laboratorio: Empaquetado de aplicaciones de Azure Virtual Desktop (AD DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure
- Una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure y con el rol Propietario o Colaborador en la suscripción de Azure
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**
- Haber completado el laboratorio **Administración de perfiles de Azure Virtual Desktop (AD DS)**
- Haber completado el laboratorio **Configuración de directivas de acceso condicional para WVD (AD DS)**

## Tiempo estimado

60 minutos

## Escenario del laboratorio

Tiene que empaquetar e implementar aplicaciones de Azure Virtual Desktop en un entorno de Active Directory Domain Services (AD DS).

## Objetivos
  
Después de completar este laboratorio, podrá:

- Preparar y crear paquetes de aplicaciones MSIX
- Implementar una imagen de conexión de aplicaciones MSIX para Azure Virtual Desktop en un entorno de Microsoft Entra DS
- Implementar la imagen de conexión de aplicaciones MSIX en Azure Virtual Desktop en un entorno de AD DS

## Archivos de laboratorio

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## Instrucciones

### Ejercicio 1: Preparar y crear paquetes de aplicaciones MSIX

Las tareas principales de este ejercicio son las siguientes:

1. Preparar la configuración de hosts de sesión de Azure Virtual Desktop
1. Implementar una máquina virtual de Azure que ejecuta Windows 10 mediante una plantilla de inicio rápido de Azure Resource Manager
1. Preparar la máquina virtual de Azure que ejecuta Windows 10 para el empaquetado MSIX
1. Generación de un certificado de firma
1. Descargar software para empaquetarlo
1. Instalar la herramienta de empaquetado MSIX
1. Crear un paquete MSIX

#### Tarea 1: Preparar la configuración de hosts de sesión de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En el equipo de laboratorio, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para iniciar las máquinas virtuales de Azure del host de sesión de Azure Virtual Desktop que vamos a usar en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en iniciarse. 

   >**Nota**: si ha habilitado PSRemoting en los hosts de sesión del grupo de recursos az140-21-RG en la primera tarea del laboratorio anterior (Implementar y administrar perfiles de AVD), puede continuar directamente con la siguiente tarea sin esperar a que se inicien las máquinas virtuales de Azure. Si no ha habilitado previamente PSRemoting en los hosts de sesión del grupo de recursos az140-21-RG, espere a que se inicien las máquinas virtuales y, a continuación, ejecute el siguiente comando.

1. Desde la sesión de PowerShell de **Cloud Shell**, ejecute lo siguiente para habilitar PowerShell Remoting en los hosts de sesión.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
#### Tarea 2: Implementar una máquina virtual de Azure que ejecuta Windows 10 mediante una plantilla de inicio rápido de Azure Resource Manager

1. Desde el equipo de laboratorio, en la ventana del explorador web en la que se muestra Azure Portal, en la barra de herramientas del panel de Cloud Shell, seleccione el icono **Cargar/Descargar archivos**; en el menú desplegable, seleccione **Cargar** y cargue los archivos **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** y**\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json** en el directorio principal de Cloud Shell.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para implementar una máquina virtual de Azure que ejecute Windows 10, que se usará para crear paquetes MSIX y unirla al dominio de Microsoft Entra DS:

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **Nota**: Espere a que la implementación se complete antes de avanzar a la siguiente tarea. Esto puede tardar unos 10 minutos. 

#### Tarea 3: Preparar la máquina virtual de Azure que ejecuta Windows 10 para el empaquetado MSIX

1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, en la lista de máquinas virtuales, seleccione la entrada **az140-cl-vm42**. Se abrirá la hoja **az140-cl-vm42**.
1. En la hoja **az140-cl-vm42**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-cl-vm42 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, inicie sesión con el nombre de usuario **wvdadmin1@adatum.com** y la contraseña que estableció al crear esta cuenta de usuario. 
1. En la sesión de Bastion a **az140-cl-vm42**, inicie **Windows PowerShell ISE** como administrador, en la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente a fin de preparar el sistema operativo para el empaquetado de MSIX:

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   ```

   > **Nota**: El último de estos cambios del registro deshabilita el Control de acceso del usuario. Esto no es técnicamente necesario, pero simplifica el proceso que se muestra en este laboratorio.

#### Tarea 4: Generación de un certificado de firma

> **Nota**: En este laboratorio se usará un certificado autofirmado. En un entorno de producción, debe usar un certificado que emita una Entidad de certificación pública o interna, según el uso previsto.

1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para generar un certificado autofirmado con el atributo Common Name establecido en **Adatum** y almacenar el certificado en la carpeta **Personal** del almacén de certificados **Equipo local**:

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. Desde la consola de**Administrador: Windows PowerShell ISE**, ejecute lo siguiente para iniciar la consola de **Certificados** destinada al almacén de certificados del Equipo local:

   ```powershell
   certlm.msc
   ```

1. En el panel de la consola de **Certificados**, expanda la carpeta **Personal**, seleccione la subcarpeta **Certificados**, haga clic con el botón derecho en el certificado **Adatum** y, en el menú contextual, seleccione **Todas las tareas** seguido de **Exportar**. Se iniciará el **Asistente para exportar certificados**. 
1. En la página **Este es el Asistente para exportar certificados** de la página **Asistente para exportar certificados**, seleccione **Siguiente**.
1. En la página **Exportar la clave privada** del **Asistente para exportar certificados**, seleccione la opción **Exportar la clave privada** y luego **Siguiente**.
1. En la página **Formato de archivo de exportación** del **Asistente para exportar certificados**, active la casilla **Exportar todas las propiedades extendidas**, desactive la casilla **Habilitar privacidad de certificado** y seleccione **Siguiente**.
1. En la página **Seguridad** del **Asistente para exportar certificados**, active la casilla **Contraseña**; en los cuadros de texto siguientes, escriba **Pa55w.rd1234** y seleccione **Siguiente**.
1. En la página **Archivo que se va a exportar** del **Asistente para exportar certificados**, en el cuadro de texto **Nombre de archivo**, seleccione **Examinar**; en el cuadro de diálogo **Guardar como**, vaya a la carpeta **C:\\Allfiles\\Labs\\04** (cree primero la carpeta); en el cuadro de texto **Nombre de archivo**, escriba **adatum.pfx** y seleccione **Guardar**.
1. De nuevo en la página **Archivo que se va a exportar** del **Asistente para exportar certificados**, asegúrese de que el cuadro de texto contiene la entrada **C:\\Allfiles\\Labs\\04\\adatum.pfx** y seleccione **Siguiente**.
1. En la página **Completing Certificate Export Wizard** (Asistente para finalizar la exportación de certificados) del **Asistente para exportar certificados**, seleccione **Finalizar** y luego **Aceptar** para confirmar la exportación correcta. 

   > **Nota**: Dado que está usando un certificado autofirmado, debe instalarlo en el almacén de certificados **Personas de confianza** en los hosts de sesión de destino.

1. Desde la consola de**Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar el certificado recién generado en el almacén de certificados **Personas de confianza** en los hosts de sesión de destino:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   $localPath = 'C:\Allfiles\Labs\04'
   ForEach ($wvdhost in $wvdhosts){
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      New-Item -ItemType Directory -Path $remotePath -Force
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### Tarea 5: Descargar software para empaquetarlo

1. En la sesión de Bastion a **az140-cl-vm42**, inicie **Microsoft Edge** y vaya a **https://github.com/microsoft/XmlNotepad**.
1. En la página **microsoft/XmlNotepad** **readme.md**, seleccione el vínculo de descarga del instalador descargable independiente y descargue los archivos de instalación comprimidos.
1. En la sesión de Bastion a **az140-cl-vm42**, inicie Explorador de archivos, vaya a la carpeta **Descargas**, abra el archivo comprimido, copie el contenido de la carpeta del archivo comprimido y péguelo en el directorio **C:\\AllFiles\\Labs\\04\\**. 

#### Tarea 6: Instalar la herramienta de empaquetado MSIX

1. En la sesión de Bastion a **az140-cl-vm42**, inicie la aplicación de **Microsoft Edge**.
1. En la aplicación de **Microsoft Store**, busque y seleccione **MSIX Packaging Tool** (Herramienta de empaquetado MSIX), en la página **MSIX Packaging Tool**(Herramienta de empaquetado MSIX), seleccione **Obtener**.
1. Cuando se le solicite, omita el inicio de sesión, espere a que se complete la instalación, seleccione **Abrir** y, en el cuadro de diálogo **Enviar datos de diagnóstico**, seleccione **Rechazar**, 

#### Tarea 7: Crear un paquete MSIX

1. En la sesión de Bastion a **az140-cl-vm42**, cambie a la ventana **Administrador: Windows PowerShell ISE** y, en la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para deshabilitar el servicio de Windows Search:

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear la carpeta que hospedará el paquete MSIX:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para quitar el flujo de datos alternativo Zone.Identifier de los archivos del instalador extraídos, que tiene un valor de "3" para indicar que se descargaron desde Internet:

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. En la sesión de Bastion a **az140-cl-vm42**, cambie a la interfaz **MSIX Packaging Tool** (Herramienta de empaquetado MSIX), en la página **Seleccionar tarea**, seleccione la entrada **Application package - Create your app package** (Paquete de aplicación: Crear el paquete de la aplicación). Se iniciará el Asistente para **crear un paquete**.
1. En la página **Seleccionar entorno** del Asistente para **crear un paquete**, asegúrese de que la opción **Create package on this computer** (Crear paquete en este equipo) está seleccionada, seleccione **Siguiente** y espere a la instalación del **controlador de la herramienta de empaquetado MSIX**.
1. En la página **Preparar equipo** del Asistente para **crear un paquete**, revise las recomendaciones. Si hay un reinicio pendiente, reinicie el sistema operativo, vuelva a iniciar sesión con la cuenta **wvdadmin1@adatum.com** y reinicie la **Herramienta de empaquetado MSIX** antes de continuar. 

   >**Nota**: Herramienta de empaquetado MSIX deshabilita temporalmente Windows Update y Windows Search. En este caso, el servicio de Windows Search ya está deshabilitado. 

1. En la página **Preparar equipo** del Asistente para **crear un paquete**, haga clic en **Siguiente**.
1. En la página **Seleccionar instalador** del Asistente para **crear un paquete**, junto al cuadro de texto **Choose the installer you want to package** (Elegir el instalador que quiere empaquetar), seleccione **Examinar**; en el cuadro de diálogo **Abrir**, vaya a la carpeta **C:\\AllFiles\\Labs\\04**, seleccione **XmlNotepadSetup.msi** y haga clic en **Abrir**. 
1. En la página **Seleccionar instalador** del Asistente para **crear un paquete**, en la lista desplegable **Signing preference** (Preferencias de firma), seleccione la entrada **Sign with a certificate (.pfx)** (Firmar con un certificado (.pfx)) junto al cuadro de texto **Buscar certificado**, seleccione **Examinar**; en el cuadro de diálogo **Abrir**, vaya a la carpeta **C:\\AllFiles\\Labs\\04**, seleccione el archivo **adatum.pfx**, haga clic en **Abrir**; en el cuadro de texto **Contraseña**, escriba **Pa55w.rd1234** y seleccione **Siguiente**.
1. En la página **Información** del Asistente para **crear un paquete**, revise la información del paquete, valide que el nombre del publicador esté establecido en **CN=Adatum** y seleccione **Siguiente**. Esto desencadenará la instalación del software descargado.
1. En la ventana **XML Notepad Setup** (Configuración del bloc de notas XML), acepte los términos del Contrato de licencia, seleccione **Instalar** y, una vez completada la instalación, active la casilla **Launch XML Notepad** (Iniciar bloc de notas XML) y seleccione **Finalizar**.
1. Cuando se le solicite, en la ventana **XML Notepad Analytics** (Análisis del Bloc de notas XML), seleccione **No**, compruebe que el Bloc de notas XML se está ejecutando, ciérrelo, vuelva al Asistente para **crear un paquete** en la ventana **MSIX Packaging Tool** (Herramienta de empaquetado MSIX) y seleccione **Siguiente**.

   > **Nota**: En este caso, no es necesario reiniciar para completar la instalación.

1. En la página **First launch tasks** (Primeras tareas de inicio) del Asistente para **crear un paquete**, revise la información proporcionada y seleccione **Siguiente**.
1. Cuando se le pida **Are you done?** (¿Ha terminado?), seleccione **Yes, move on** (Sí, continuemos).
1. En la página **Services report** (Informe de servicios) del Asistente para **crear un paquete**, compruebe que no aparece ningún servicio y seleccione **Siguiente**.
1. En la página **Crear paquete** del Asistente para **crear un paquete**, en el cuadro de texto **Ubicación en la que desea guardar**, escriba **C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix** y haga clic en **Crear**.
1. En el cuadro de diálogo **Package successfully created** (Paquete creado correctamente), anote la ubicación del paquete guardado y seleccione **Cerrar**.
1. Cambie a la ventana Explorador de archivos, vaya a la carpeta **C:\\Allfiles\\Labs\\04\\XmlNotepad** y compruebe que contiene los archivos *.msix y *.xml.
1. Copie el archivo **XmlNotepad.msix** en la carpeta **C:\\Allfiles\\Labs\\04**.


### Ejercicio 2: Implementar una imagen de conexión de aplicaciones MSIX para Azure Virtual Desktop en el entorno de Microsoft Entra DS

Las tareas principales de este ejercicio son las siguientes:

1. Habilitar Hyper-V en las máquinas virtuales de Azure que ejecutan Windows 10 Enterprise Edition
1. Crear una imagen de conexión de aplicaciones MSIX

#### Tarea 1: Habilitar Hyper-V en las máquinas virtuales de Azure que ejecutan Windows 10 Enterprise Edition

1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente a fin de preparar los hosts de Azure Virtual Desktop de destino para la conexión de aplicaciones MSIX: 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar Hyper-V y sus herramientas de administración, incluido el módulo de PowerShell de Hyper-V en los hosts de Azure Virtual Desktop:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. Cuando se le pida que reinicie el sistema operativo de destino, seleccione **Sí**.
1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar Hyper-V y sus herramientas de administración, incluido el módulo de PowerShell de Hyper-V en el equipo local:

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Una vez completada la instalación de los componentes de Hyper-V, seleccione **Sí** para reiniciar el sistema operativo. Después del reinicio, vuelva a iniciar sesión con el nombre de usuario **wvdadmin1@adatum.com** y la contraseña que estableció al crear esta cuenta de usuario.

#### Tarea 2: Crear una imagen de conexión de aplicaciones MSIX

1. En la sesión de Bastion a **az140-cl-vm42**, inicie **Microsoft Edge**, vaya a **https://aka.ms/msixmgr**. Esto descargará automáticamente el archivo **msixmgr.zip** (el archivo de herramientas mgr MSIX) en la carpeta **Descargas**.
1. En Explorador de archivos, vaya a la carpeta **Descargas**, abra el archivo comprimido y copie el contenido de la carpeta **x64** (incluida la carpeta) en la carpeta **C:\\AllFiles\\Labs\\04**. 
1. En la sesión de Bastion a **az140-cl-vm42**, inicie **Windows PowerShell ISE** como administrador y, en la consola de **Administrador: Windows PowerShell script ISE**, ejecute lo siguiente para crear el archivo VHD que servirá como imagen de conexión de aplicaciones MSIX:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. Desde el panel del script de**Administrador: Windows PowerShell ISE**, ejecute lo siguiente para montar el archivo VHD recién creado:

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. Desde el panel del script de**Administrador: Windows PowerShell script de ISE**, ejecute lo siguiente para inicializar el disco, crear una partición, formatearla y asignarle la primera letra de unidad disponible:

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

   > **Nota**: Si aparece una ventana emergente que le pide que formatee la unidad F:, seleccione **Cancelar**.

1. Desde el panel del script de**Administrador: Windows PowerShell script de ISE**, ejecute lo siguiente para crear una estructura de carpetas que hospedará los archivos MSIX y desempaquetará en él el paquete MSIX que creó en la tarea anterior:

   ```powershell
   $appName = 'XmlNotepad'
   New-Item -ItemType Directory -Path "$($partition.DriveLetter):\Apps" -Force
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination "$($partition.DriveLetter):\Apps" -applyacls
   ```

1. En la sesión de Bastion a **az140-cl-vm42**, en Explorador de archivos, vaya a la carpeta **F:\\Apps** y revise su contenido. Si se le pide que obtenga acceso a la carpeta, seleccione **Continuar**.
1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell script ISE**, ejecute lo siguiente para desmontar el archivo VHD que servirá como imagen de MSIX:

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### Ejercicio 3: Implementación de conexión de aplicaciones MSIX en hosts de sesión de Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Configurar grupos de Active Directory que incluyan hosts de Azure Virtual Desktop
1. Configurar el recurso compartido de Azure Files para la conexión de aplicaciones MSIX
1. Montar y registrar de a imagen de conexión de aplicaciones MSIX en hosts de sesión de Azure Virtual Desktop
1. Publicar de aplicaciones MSIX en un grupo de aplicaciones
1. Validar la función de la conexión de aplicaciones MSIX

#### Tarea 1: Configurar grupos de Active Directory que incluyan hosts de Azure Virtual Desktop

1. Cambie al equipo de laboratorio; en el explorador web donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione **az140-dc-vm11**.
1. En la hoja **az140-dc-vm11**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-dc-vm11 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Bastion a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear un objeto de grupo de AD DS que se sincronizará con el inquilino de Microsoft Entra usado en este laboratorio:

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **Nota**: Usará este grupo para conceder permisos de hosts de Azure Virtual Desktop al recurso compartido de archivos **az140-42-msixvhds**.

1. Desde el panel del script de**Administrador: Windows PowerShell consola ISE**, ejecute lo siguiente para agregar miembros a los grupos que ha creado en el paso anterior:

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. Desde el panel del script de**Administrador: Windows PowerShell ISE**, ejecute lo siguiente para reiniciar los servidores que son miembros del grupo "az140-hosts-42-p1":

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **Nota**: Este paso garantiza que el cambio de pertenencia a grupos surta efecto. 

1. En la sesión de Bastion a **az140-dc-vm11**, en el menú **Inicio**, expanda la carpeta **Microsoft Entra Connect** y seleccione **Microsoft Entra Connect**.
1. En la página **Bienvenido a Microsoft Entra Connect** de la ventana **Microsoft Entra Connect**, seleccione **Configurar**.
1. En la página **Tareas adicionales** de la ventana **Microsoft Entra Connect**, seleccione **Personalizar las opciones de sincronización** y luego, **Siguiente**.
1. En la página **Conectar a Microsoft Entra** de la ventana **Microsoft Entra Connect**, autentíquese mediante el nombre principal de usuario de la cuenta de usuario **aadsyncuser** que identificó anteriormente en esta tarea con la contraseña que estableció al crear esta cuenta de usuario.
1. En la página **Conectar sus directorios** de la ventana **Microsoft Entra Connect**, seleccione **Siguiente**.
1. En la página **Filtrado de dominios y unidades organizativas** de la ventana **Microsoft Entra Connect**, asegúrese de que la opción **Sincronizar los dominios y las unidades organizativas seleccionados** está seleccionada, expanda el nodo **adatum.com**, seleccione la casilla situada junto a la unidad organizativa **WVDInfra** (deje el resto de casillas seleccionadas sin cambios) y seleccione **Siguiente**.
1. En la página **Características opcionales** de la ventana **Microsoft Entra Connect**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la página **Listo para configurar** de la ventana **Microsoft Entra Connect**, asegúrese de que la casilla **Inicie el proceso de sincronización cuando se complete la configuración** está seleccionada y seleccione **Configurar**.
1. Revise la información de la página **Configuración completada** y seleccione **Salir** para cerrar la ventana **Microsoft Entra Connect**.
1. En la sesión de Bastion a **az140-dc-vm11**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Cuando se le solicite, inicie sesión con las credenciales de Microsoft Entra de la cuenta de usuario con el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure que estamos usando en este laboratorio.
1. En la sesión de Bastion a **az140-dc-vm11**, en la ventana de Microsoft Edge donde se muestra Azure Portal, busque y seleccione **Azure Active Directory** para ir al inquilino de Microsoft Entra asociado a la suscripción de Azure que estamos usando en este laboratorio.
1. En la barra de menús vertical izquierda de la hoja de Azure Active Directory, en la sección **Administrar**, seleccione **Grupos**. 
1. En la hoja **Grupos | Todos los grupos**, en la lista de grupos, seleccione la entrada **az140-hosts-42-p1**.

   > **Nota**: Es posible que tenga que actualizar la página para que se muestre el grupo.

1. En la hoja **az140-hosts-42-p1**, en la barra de menús vertical del lado izquierdo, en la sección **Administrar**, haga clic en **Miembros**.
1. En la hoja **az140-hosts-42-p1 | Miembros**, compruebe que la lista de **Miembros directos** incluye los tres hosts del grupo de Azure Virtual Desktop que agregó al grupo anteriormente en esta tarea.

#### Tarea 2: Configurar el recurso compartido de Azure Files para la conexión de aplicaciones MSIX

1. En el equipo de laboratorio, cambie a la sesión de Bastion a **az140-cl-vm42**.
1. En la sesión de Bastion a **az140-cl-vm42**, inicie Microsoft Edge en el modo InPrivate, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión proporcionando credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.

   > **Nota**: Asegúrese de usar el modo InPrivate de Microsoft Edge.

1. En la sesión de Bastion a **az140-cl-vm42**, en la ventana de Microsoft Edge en la que se muestra Azure Portal, busque y seleccione **Cuentas de almacenamiento** y, en la hoja **Cuentas de almacenamiento**, seleccione la cuenta de almacenamiento que configuró para hospedar los perfiles de usuario.

   > **Nota**: Esta parte del laboratorio depende de completar el laboratorio **Administración de perfiles de Azure Virtual Desktop (AD DS)** o **Administración de perfiles de Azure Virtual Desktop (Microsoft Entra DS)**

   > **Nota**: En escenarios de producción, debe plantearse usar una cuenta de almacenamiento independiente. Esto requeriría configurar esa cuenta de almacenamiento para la autenticación de Microsoft Entra DS, que ya implementó para la cuenta de almacenamiento que hospeda perfiles de usuario. Está usando la misma cuenta de almacenamiento para minimizar los pasos duplicados en laboratorios individuales.

1. En la hoja de la cuenta de almacenamiento, en el menú vertical del lado izquierdo, seleccione **Control de acceso (IAM)**.
1. En la hoja **Control de acceso (IAM)** de la cuenta de almacenamiento, seleccione **+ Agregar** y, en el menú desplegable, seleccione **Agregar asignación de roles**. 
1. En el panel **Agregar asignación de roles**, especifique la configuración siguiente y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Role|**Colaborador elevado de recursos compartidos de SMB de datos de archivos de Storage**|
   |Asignar acceso a|**Usuario, grupo o entidad de servicio**|
   |Seleccionar|**az140-wvd-admins**|

   > **Nota**: El grupo **az140-wvd-admins** contiene la cuenta de usuario **wvdadmin1**, que usará para configurar los permisos del recurso compartido. 

1. Repita los dos pasos anteriores para configurar las asignaciones de roles siguientes:

   |Configuración|Valor|
   |---|---|
   |Role|**Colaborador elevado de recursos compartidos de SMB de datos de archivos de Storage**|
   |Asignar acceso a|**Usuario, grupo o entidad de servicio**|
   |Seleccionar|**az140-hosts-42-p1**|

   |Configuración|Valor|
   |---|---|
   |Role|**Lector de recursos compartidos de SMB de datos de archivos de Storage**|
   |Asignar acceso a|**Usuario, grupo o entidad de servicio**|
   |Seleccionar|**az140-wvd-users**|

   > **Nota**: Los usuarios y hosts de Azure Virtual Desktop necesitan al menos acceso de lectura al recurso compartido de archivos.

1. En la hoja Cuenta de almacenamiento, en el menú vertical situado en el lado izquierdo, en la sección **Almacenamiento de datos**, seleccione **Recursos compartidos de archivos** y, después, **+ Recurso compartido de archivos**.
1. En la hoja **Nuevo recurso compartido**, especifique las opciones de configuración siguientes y seleccione **Crear** (deje las demás con los valores predeterminados):

   |Configuración|Value|
   |---|---|
   |Nombre|**az140-42-msixvhds**|

1. En la ventana de Microsoft Edge donde se muestra Azure Portal, en la lista de recursos compartidos de archivos, seleccione el recurso compartido de archivos recién creado. 

1. En la sesión de Bastion a **az140-cl-vm42**, inicie **Símbolo del sistema** y, desde la ventana **Símbolo del sistema**, ejecute lo siguiente para asignar una unidad al recurso compartido **az140-42-msixvhds** (reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento) y compruebe que el comando se completa correctamente:

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. En la sesión de Bastion a **az140-cl-vm42**, desde la ventana **Símbolo del sistema**, ejecute lo siguiente para conceder los permisos NTFS necesarios a las cuentas de equipo de los hosts de sesión:

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**Nota**: También puede establecer estos permisos mediante Explorador de archivos durante el inicio de sesión como **wvdadmin1\@adatum.com**. 

   >**Nota**: A continuación validará la función de la conexión de aplicaciones MSIX.

1. En la sesión de Bastion a **az140-cl-vm42**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para copiar el archivo VHD que creó en el ejercicio anterior al recurso compartido de Azure Files que creó anteriormente en este ejercicio:

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### Tarea 3: Montar y registrar de a imagen de conexión de aplicaciones MSIX en hosts de sesión de Azure Virtual Desktop

1. En la sesión de Bastion a **az140-cl-vm42**, en la ventana de Microsoft Edge en la que se muestra Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, en el menú vertical de la izquierda, en la sección **Administrar**, seleccione **Grupos de hosts**.
1. En la hoja **Azure Virtual Desktop \| Grupos de hosts**, en la lista de grupos de hosts, seleccione la entrada **az140-21-hp1**.
1. En la hoja **az140-21-hp1 \| Propiedades**, en el menú vertical del lado izquierdo, en la sección **Administrar**, seleccione **Paquetes MSIX**.
1. En la hoja **az140-21-hp1 \| MSIX packages** (Paquetes MSIX), haga clic en **+ Agregar**.
1. En la hoja **Add MSIX package** (Agregar paquete MSIX), en el cuadro de texto **MSIX image path** (Ruta de acceso de la imagen MSIX), escriba la ruta de acceso al archivo **XmlNotepad.vhd** con el formato `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd` (reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento que hospeda el recurso compartido de archivos **az140-42-msixvhds** ) y haga clic en **Agregar**.
1. En la hoja **Add MSIX package** (Agregar paquete MSIX), especifique la configuración siguiente y haga clic en **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Ruta de acceso de la imagen de MSIX|**\\\\\<storage-account-name\>.file.core.windows.net\\az140-42-msixvhds\\XmlNotepad.vhd**, donde el marcador de posición `<storage-account-name>` designa el nombre de la cuenta de almacenamiento que hospeda el recurso compartido de archivos **az140-42-msixvhds**|
   |Paquete MSIX|el nombre generado durante la creación del paquete.|
   |Nombre para mostrar|**Bloc de notas XML**|
   |Tipo de registro|**A petición**|
   |Estado|**Active**|

#### Tarea 4: Publicar de aplicaciones MSIX en un grupo de aplicaciones

> **Nota**: Publicará la aplicación MSIX en la aplicación remota y en el grupo de aplicaciones de escritorio. 

1. En la sesión de Bastion a **az140-cl-vm42**, en la ventana de Microsoft Edge en la que se muestra Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, en el menú vertical de la izquierda, en la sección **Administrar**, seleccione **Grupos de aplicaciones**.
1. En la hoja **Azure Virtual Desktop \| Grupos de aplicaciones**, seleccione la entrada del grupo de aplicaciones **az140-21-hp1-Utilities-RAG**.
1. En la hoja **az140-21-hp1-Utilities-RAG**, en el menú vertical del lado izquierdo, en la sección **Administrar**, seleccione **Aplicaciones**. 
1. En la hoja **az140-21-hp1-Utilities-RAG \| Aplicaciones**, haga clic en **+ Agregar**.
1. En la hoja **Agregar aplicaciones**, especifique la configuración siguiente y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Paquete MSIX**|
   |Paquete MSIX|el nombre que representa el paquete incluido en la imagen|
   |Aplicación MSIX|**XMLNOTEPAD**|
   |Nombre de la aplicación|**Bloc de notas XML**|
   |Nombre para mostrar|**Bloc de notas XML**|
   |Descripción|**Bloc de notas XML**|
   |Ruta de icono|**C:\\Archivos de programa\\WindowsApps\\XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8\\VFS\\ProgramFilesX86\\LovettSoftware\\XmlNotepad\\XmlNotepad.exe**|
   |Índice de icono|**0**|

1. Vuelva a la hoja **Azure Virtual Desktop \| Grupos de aplicaciones** y seleccione la entrada del grupo de aplicaciones **az140-21-hp1-DAG**.
1. En la hoja **az140-21-hp1-DAG**, en el menú vertical del lado izquierdo, en la sección **Administrar**, seleccione **Aplicaciones**. 
1. En la hoja **az140-21-hp1-DAG \| Aplicaciones**, haga clic en **+ Agregar**.
1. En la hoja **Agregar aplicaciones**, especifique la configuración siguiente y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Paquete MSIX**|
   |Paquete MSIX|el nombre que representa el paquete incluido en la imagen|
   |Nombre de la aplicación|**Bloc de notas XML**|
   |Nombre para mostrar|**Bloc de notas XML**|
   |Descripción|**Bloc de notas XML**|

#### Tarea 5: Validar la función de la conexión de aplicaciones MSIX

1. En la sesión de Bastion a **az140-cl-vm42**, inicie Microsoft Edge y vaya a la [página de descarga del cliente de escritorio de Windows](https://go.microsoft.com/fwlink/?linkid=2068602); cuando se complete la descarga, seleccione **Abrir archivo** para iniciar su instalación. En la página **Ámbito de instalación** del asistente de **Instalación de Escritorio remoto**, seleccione la opción **Instalar para todos los usuarios de esta máquina** y haga clic en **Instalar**. 
1. Una vez completada la instalación, asegúrese de que la casilla **Iniciar Escritorio remoto cuando se cierre la instalación** esté activada y haga clic en **Finalizar** para iniciar el cliente de Escritorio remoto.
1. En la ventana de cliente **Escritorio remoto**, seleccione **Suscribirse** y, cuando se le solicite, inicie sesión con el nombre principal de usuario **aduser1** y la contraseña que estableció al crear esta cuenta de usuario. 
1. Si se le solicita, en la ventana **Mantener sesión iniciada en todas las aplicaciones**, desactive la casilla **Permitir que mi organización administre el dispositivo** y haga clic en **No, iniciar sesión solo en esta aplicación**.
1. En la ventana de cliente **Escritorio remoto**, en la sección **az140-21-ws1**, haga doble clic en el icono **Bloc de notas XML**; cuando se le solicite, proporcione la contraseña y compruebe que el Bloc de notas XML se inicia correctamente.


### Ejercicio 4: Detención y desasignación de las máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detener y desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignaremos las máquinas virtuales de Azure aprovisionadas en este laboratorio para reducir al mínimo los cargos de proceso correspondientes.

#### Tarea 1: Desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel de Cloud Shell, ejecute lo siguiente para mostrar todas las máquinas virtuales de Azure que hemos creado y usado en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. Desde la sesión de PowerShell, en el panel de Cloud Shell, ejecute lo siguiente para detener y desasignar todas las máquinas virtuales de Azure que hemos creado y usado en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.
