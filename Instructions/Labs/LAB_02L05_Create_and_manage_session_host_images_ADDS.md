---
lab:
  title: "Laboratorio: Creación y administración de imágenes de host de sesión (AD\_DS)"
  module: 'Module 2: Implement a WVD Infrastructure'
---

# <a name="lab---create-and-manage-session-host-images-ad-ds"></a>Laboratorio: Creación y administración de imágenes de host de sesión (AD DS)
# <a name="student-lab-manual"></a>Manual de laboratorio para alumnos

## <a name="lab-dependencies"></a>Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Azure AD con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Azure AD asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**

## <a name="estimated-time"></a>Tiempo estimado

60 minutos

## <a name="lab-scenario"></a>Escenario del laboratorio

Debe crear y administrar imágenes de host de Azure Virtual Desktop en un entorno de Active Directory Domain Services (AD DS).

## <a name="objectives"></a>Objetivos
  
Después de completar este laboratorio, podrá:

- Crear y administrar imágenes de host de sesión

## <a name="lab-files"></a>Archivos de laboratorio

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## <a name="instructions"></a>Instructions

### <a name="exercise-1-create-and-manage-session-host-images"></a>Ejercicio 1: Creación y administración de imágenes de host de sesión
  
Las tareas principales de este ejercicio son las siguientes:

1. Preparar la configuración de una imagen de host de Azure Virtual Desktop
1. Implementación de Azure Bastion
1. Configurar una imagen de host de Azure Virtual Desktop
1. Crear una imagen de host de Azure Virtual Desktop
1. Aprovisionar un grupo de hosts de Azure Virtual Desktop mediante la imagen personalizada

#### <a name="task-1-prepare-for-configuration-of-a-azure-virtual-desktop-host-image"></a>Tarea 1: Preparar la configuración de una imagen de host de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En Azure Portal, seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda para abrir el panel de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **PowerShell**. 
1. En el equipo de laboratorio, en el explorador web en el que se muestra Azure Portal, desde la sesión de PowerShell en el panel de Cloud Shell, ejecute lo siguiente para crear un grupo de recursos que incluirá la imagen de host de Azure Virtual Desktop:

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. En Azure Portal, en la barra de herramientas del panel de Cloud Shell, seleccione el icono **Cargar/Descargar archivos**; en el menú desplegable, seleccione **Cargar** y cargue los archivos **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json** y **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json** en el directorio principal de Cloud Shell.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para implementar una máquina virtual de Azure que ejecute Windows 10, que actuará como cliente de Azure Virtual Desktop en la subred recientemente creada:

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **Nota**: Espere a que la implementación se complete antes de avanzar al siguiente ejercicio. La implementación puede tardar unos 10 minutos.

#### <a name="task-2-deploy-azure-bastion"></a>Tarea 2: Implementación de Azure Bastion 

> **Nota**: Azure Bastion permite la conexión a las máquinas virtuales de Azure sin puntos de conexión públicos que implementamos en la tarea anterior de este ejercicio, al tiempo que proporciona protección contra vulnerabilidades de seguridad de fuerza bruta que tienen como destino las credenciales de nivel de sistema operativo.

> **Nota**: Asegúrese de que el explorador tiene habilitada la función emergente.

1. En la ventana del explorador donde se muestra Azure Portal, abra otra pestaña y, en la pestaña del explorador, vaya a Azure Portal.
1. En Azure Portal, seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda para abrir el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para agregar una subred denominada **AzureBastionSubnet** a la red virtual **az140-25-vnet** que creamos anteriormente en este ejercicio:

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cierre el panel de Cloud Shell.
1. En Azure Portal, busque y seleccione **Bastions** y, en la hoja **Bastions**, seleccione **+ Crear**.
1. En la pestaña **Básico** de la hoja **Crear una instancia de Bastion**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Value|
   |---|---|
   |Subscription|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az140-25-RG**|
   |Nombre|**az140-25-bastion**|
   |Region|la misma región de Azure en la que se han implementado los recursos en las tareas anteriores de este ejercicio.|
   |Nivel|**Basic**|
   |Virtual network|**az140-25-vnet**|
   |Subnet|**AzureBastionSubnet (10.25.254.0/24)**|
   |Dirección IP pública|**Crear nuevo**|
   |Nombre de la IP pública|**az140-25-vnet-ip**|

1. En la pestaña **Revisar y crear** de la hoja **Crear una instancia de Bastion**, seleccione **Crear**.

   > **Nota**: Espere a que la implementación se complete antes de avanzar al siguiente ejercicio. La implementación puede tardar unos 5 minutos.

#### <a name="task-3-configure-a-azure-virtual-desktop-host-image"></a>Tarea 3: Configurar una imagen de host de Azure Virtual Desktop

1. En Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, haga clic en **az140-25-vm0**.
1. En la hoja **az140-25-vm0**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-25-vm0 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Value|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

   > **Nota**: Para empezar, instale archivos binarios de FSLogix.

1. En la sesión de Escritorio remoto a **az140-25-vm0**, inicie **Windows PowerShell ISE** como administrador.
1. En la sesión de Escritorio remoto a **az140-25-vm0**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente a fin de crear una carpeta que usará como ubicación temporal para la configuración de la imagen:

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. En la sesión de Escritorio remoto a **az140-25-vm0**, inicie Microsoft Edge, vaya a la [página de descarga de FSLogix](https://aka.ms/fslogix_download), descargue los archivos binarios de instalación comprimidos de FSLogix en la carpeta **C:\\Allfiles\\Labs\\02** y, desde Explorador de archivos, extraiga la subcarpeta **x64** en la misma carpeta.
1. En la sesión de Escritorio remoto a **az140-25-vm0**, cambie a la ventana **Administrador: Windows PowerShell ISE** y, en la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para realizar la instalación por máquina de OneDrive:

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **Nota**: Espere a que termine la instalación. Esto puede tardar 1 minuto. Si la instalación desencadena un reinicio, vuelva a conectarse a **az140-25-vm0**.

   > **Nota**: A continuación, recorrerá la instalación y configuración de Microsoft Teams (con fines de aprendizaje, puesto que Teams ya está presente en la imagen usada para este laboratorio).

1. En la sesión de Escritorio remoto a **az140-25-vm0**, haga clic con el botón derecho en **Inicio**; en el menú contextual, seleccione **Ejecutar**; en el cuadro de diálogo **Ejecutar**, en el cuadro de texto **Abrir**, escriba **cmd** y presione la tecla **Entrar** para iniciar el **Símbolo del sistema**.
1. En la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute lo siguiente para preparar la instalación por máquina de Microsoft Teams:

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. En la sesión de Escritorio remoto a **az140-25-vm0**, en Microsoft Edge, vaya a [la página de descarga de Microsoft Visual C++ Redistributable](https://aka.ms/vs/16/release/vc_redist.x64.exe) y guarde **VC_redist.x64** en la carpeta **C:\\Allfiles\\Labs\\02**.
1. En la sesión de Escritorio remoto a **az140-25-vm0**, cambie a la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute lo siguiente para realizar la instalación de Microsoft Visual C++ Redistributable:

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. En la sesión de Escritorio remoto a **az140-25-vm0**, en Microsoft Edge, vaya a la página de documentación titulada [Implementación de la aplicación de escritorio de Teams en la máquina virtual](https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm), haga clic en el vínculo **versión de 64 bits** y, cuando se le solicite, guarde el archivo **Teams_windows_x64.msi** en la carpeta **C:\\Allfiles\\Labs\\02**.
1. En la sesión de Escritorio remoto a **az140-25-vm0**, cambie a la ventana **Administrador: C:\windows\system32\cmd.exe** y, desde el símbolo del sistema, ejecute lo siguiente para realizar la instalación por máquina de Microsoft Teams:

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **Nota**: El instalador admite los parámetros ALLUSER=1 y ALLUSERS=1. El parámetro ALLUSER=1 está pensado para la instalación por máquina en entornos de VDI. El parámetro ALLUSERS=1 se puede usar en entornos de VDI y de otro tipo. 
   > **Nota**: Si encuentra un error que indica que **Ya hay instalada otra versión del producto**, siga los pasos siguientes: Vaya a **Panel de control > Programas > Programas y características**, haga clic con el botón derecho en el programa **Instalador a nivel de todo el equipo de Teams** y seleccione **Desinstalar**. Continúe con la eliminación del programa y vuelva a realizar el paso 13 anterior. 

1. En la sesión de Escritorio remoto a **az140-25-vm0-0**, inicie **Windows PowerShell ISE** como administrador y, en la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para instalar Microsoft Edge Chromium (con fines de aprendizaje, puesto que Edge ya está presente en la imagen usada para este laboratorio):

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **Nota**: Espere a que termine la instalación. Esto puede tardar unos 2 minutos.

   > **Nota**: Al trabajar en un entorno de varios idiomas, es posible que tenga que instalar paquetes de idioma. Para obtener más información sobre este procedimiento, consulte el artículo de Microsoft Docs [Incorporación de paquetes de idioma a una imagen de Windows 10 multisesión](https://docs.microsoft.com/en-us/azure/virtual-desktop/language-packs).

   > **Nota**: A continuación, deshabilitará Actualizaciones automáticas de Windows y el Sensor de almacenamiento, configurará el redireccionamiento de zona horaria y configurará la recopilación de telemetría. En general, primero deberá aplicar todas las actualizaciones actuales. En este laboratorio, omitirá este paso para minimizar su duración.

1. En la sesión de Escritorio remoto a **az140-25-vm0**, cambie a la ventana **Administrador: C:\windows\system32\cmd.exe** y, desde el símbolo del sistema, ejecute lo siguiente para deshabilitar Actualizaciones automáticas:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. En la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute lo siguiente para deshabilitar Sensor de almacenamiento:

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. En la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute lo siguiente para configurar el redireccionamiento de zona horaria:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. En la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute lo siguiente para deshabilitar la recopilación del concentrador de comentarios de datos de telemetría:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. En la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute lo siguiente para eliminar la carpeta temporal que creó anteriormente en esta tarea:

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. En la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute la utilidad Liberador de espacio en disco y haga clic en **Aceptar** una vez completada:

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### <a name="task-4-create-a-azure-virtual-desktop-host-image"></a>Tarea 4: Crear una imagen de host de Azure Virtual Desktop

1. En la sesión de Escritorio remoto a **az140-25-vm0**, en la ventana **Administrador: C:\windows\system32\cmd.exe**, desde el símbolo del sistema, ejecute la utilidad sysprep para preparar el sistema operativo a fin de generar una imagen y apagarla automáticamente:

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown
   ```

   > **Nota**: Espere a que se complete el proceso sysprep. Esto puede tardar unos 2 minutos. Esto apagará automáticamente el sistema operativo. 

1. Desde el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione **az140-25-vm0**.
1. En la hoja **az140-25-vm0**, en la barra de herramientas situada encima de la sección **Información esencial**, haga clic en **Actualizar**, compruebe que el **Estado** de la máquina virtual de Azure ha cambiado a **Detenido**, haga clic en **Detener** y, cuando se le pida confirmación, haga clic en **Aceptar** para realizar la transición de la máquina virtual de Azure al estado **Detenido (desasignado).**
1. En la hoja **az140-25-vm0**, compruebe que el **Estado** de la máquina virtual de Azure ha cambiado al estado **Detenido (desasignado)** y, en la barra de herramientas, haga clic en **Capturar**. Se mostrará automáticamente la hoja **Crear una imagen**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear una imagen**, especifique la configuración siguiente:

   |Configuración|Valor|
   |---|---|
   |Compartir una imagen en la galería de procesos de Azure|**Sí, compartirla en una galería como una versión de imagen**|
   |Eliminar automáticamente esta máquina virtual después de crear la imagen|casilla desactivada|
   |Galería de procesos de Azure objetivo|el nombre de una nueva galería **az14025imagegallery**|
   |Estado del sistema operativo|**Generalizada**|

1. En la pestaña **Aspectos básicos** de la hoja **Crear una imagen**, debajo del cuadro de texto **Target VM image definition** (Definición de la imagen de máquina virtual de destino), haga clic en **Crear**.
1. En **Create a VM image definition** (Crear la definición de una imagen de máquina virtual), especifique la configuración siguiente y haga clic en **Aceptar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de definición de la imagen de máquina virtual|**az140-25-host-image**|
   |Publisher|**MicrosoftWindowsDesktop**|
   |Oferta|**office-365**|
   |SKU|**20h1-evd-o365pp**|

1. De nuevo en la pestaña **Aspectos básicos** de la hoja **Crear una imagen**, especifique la configuración siguiente y haga clic en **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Número de la versión|**1.0.0**|
   |Excluir de la versión más reciente|casilla desactivada|
   |Fecha final del ciclo de vida|un año antes de la fecha actual|
   |Recuento de réplicas predeterminado|**1**|
   |Número de réplicas de la región de destino|**1**|
   |Tipo de cuenta de almacenamiento|**LRS de SSD Premium**|

1. En la pestaña **Revisar y crear** del panel **Crear una imagen**, haga clic en **Crear**.

   > **Nota**: Espere a que la implementación se complete. Esto puede tardar unos 20 minutos.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busque y seleccione **Azure compute galleries** (Galerías de proceso de Azure) y, en la hoja **Galerías de proceso de Azure**, seleccione la entrada **az14025imagegallery** y, en la hoja ****az14025imagegallery****, compruebe la presencia de la entrada **az140-25-host-image** que representa la imagen recién creada.

#### <a name="task-5-provision-a-azure-virtual-desktop-host-pool-by-using-a-custom-image"></a>Tarea 5: Aprovisionar un grupo de hosts de Azure Virtual Desktop mediante una imagen personalizada

1. En el equipo de laboratorio, en Azure Portal, utilice el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar **Redes virtuales** y navegar a ellas y, en la hoja **Redes virtuales**, seleccione **az140-adds-vnet11**. 
1. En la hoja **az140-adds-vnet11**, seleccione **Subredes**, y en la hoja **Subredes**, seleccione **+ Subred**. Después, en la hoja **Agregar subred**, especifique la siguiente configuración (deje todas las demás opciones con sus valores predeterminados) y haga clic en **Guardar**:

   |Configuración|Value|
   |---|---|
   |Nombre|**hp4-Subnet**|
   |Intervalo de direcciones de subred|**10.0.4.0/24**|

1. En el equipo de laboratorio, en Azure Portal, en la ventana del explorador web que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop**; en la hoja **Azure Virtual Desktop**, seleccione **Grupos de hosts** y, en la hoja **Azure Virtual Desktop \| Grupos de hosts**, seleccione **+Crear**. 
1. En la pestaña **Aspectos básicos** de la hoja **Crear un grupo de hosts**, especifique la siguiente configuración y seleccione **Siguiente: Máquinas virtuales >** :

   |Configuración|Value|
   |---|---|
   |Subscription|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az140-25-RG**|
   |Nombre del grupo de hosts|**az140-25-hp4**|
   |Location|nombre de la región de Azure en la que implementó recursos en el primer ejercicio de este laboratorio|
   |Entorno de validación|**No**|
   |Tipo de grupo de hosts|**Agrupado**|
   |Límite máximo de sesión|**50**|
   |Algoritmo de equilibrio de carga|**Con prioridad a la amplitud**|

1. En la pestaña **Máquinas virtuales** de la hoja **Crear un grupo de hosts**, especifique la configuración siguiente:

   |Configuración|Valor|
   |---|---|
   |Agregar Máquinas virtuales de Azure|**Sí**|
   |Resource group|**El valor predeterminado es el mismo que el del grupo de hosts**|
   |Prefijo de nombre|**az140-25-p4**|
   |Ubicación de la máquina virtual|nombre de la región de Azure en la que implementó recursos en el primer ejercicio de este laboratorio|
   |Opciones de disponibilidad|No se requiere redundancia de la infraestructura|
   |Tipo de imagen|**Galería**|

1. En la pestaña **Máquinas virtuales** de la hoja **Crear un grupo de hosts**, directamente debajo de la lista desplegable **Imagen**, haga clic en el vínculo **Ver todas las imágenes**.
1. En la hoja **Seleccionar una imagen**, haga clic en la pestaña **Mis elementos**, haga clic en **Imágenes compartidas** y, en la lista de imágenes compartidas, seleccione **az140-25-host-image**. 
1. De nuevo en la pestaña **Máquinas virtuales** de la hoja **Crear un grupo de hosts**, especifique la configuración siguiente y seleccione **Siguiente: Área de trabajo >** .

   |Configuración|Valor|
   |---|---|
   |Tamaño de la máquina virtual|**Estándar D2s v3**|
   |Número de VM|**1**|
   |Tipo de disco del sistema operativo|**SSD estándar**|
   |Virtual network|**az140-adds-vnet11**|
   |Subnet|**hp4-Subnet (10.0.4.0/24)**|
   |Grupo de seguridad de red|**Basic**|
   |Puertos de entrada públicos|**Sí**|
   |Puertos de entrada que se van a permitir|**RDP**|
   |UPN de unión a un dominio de AD|**student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|
   |Especifique el nombre de dominio o la unidad|**Sí**|
   |Dominio al que desea unirse|**adatum.com**|
   |Ruta de acceso de unidad organizativa|**OU=WVDInfra,DC=adatum,DC=com**|
   |Nombre de usuario|Estudiante|
   |Contraseña|Pa55w.rd1234|
   |Confirmar contraseña|Pa55w.rd1234|

1. En la pestaña **Área de trabajo** de la hoja **Crear un grupo de hosts**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicación de escritorio|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear un grupo de hosts**, seleccione **Crear**.

   > **Nota**: Espere a que la implementación se complete. Esto puede tardar unos 10 minutos.
   > 
   > **Nota**: Si se produce un error en la implementación debido a que se alcanza el límite de cuota, realice los pasos descritos en el primer laboratorio para solicitar automáticamente el aumento de cuota del límite de las máquinas virtuales D2sv3 estándar a 30.

   > **Nota**: Siguiendo la implementación de hosts basada en imágenes personalizadas, debe considerar la posibilidad de ejecutar la herramienta de optimización Virtual Desktop, disponible en [su repositorio de GitHub](https://github.com/The-Virtual-Desktop-Team/).


### <a name="exercise-2-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>Ejercicio 2: Detención y desasignación de máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detener y desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignará las máquinas virtuales de Azure aprovisionadas en este laboratorio para minimizar los cargos de proceso correspondientes.

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>Tarea 1: Desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para enumerar todas las máquinas virtuales de Azure creadas en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. Desde la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para detener y desasignar todas las máquinas virtuales de Azure que creó en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.
