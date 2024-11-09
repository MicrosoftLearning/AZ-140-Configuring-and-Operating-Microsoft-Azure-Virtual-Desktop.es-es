---
lab:
  title: "Laboratorio: Implementación de hosts y grupos de hosts mediante plantillas de Azure Resource Manager (AD\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratorio: Implementación de hosts y grupos de hosts mediante plantillas de Azure Resource Manager
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Microsoft Entra asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**
- Haber completado el laboratorio **Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (AD DS)**

## Tiempo estimado

45 minutos

## Escenario del laboratorio

Debe automatizar la implementación de grupos de hosts y hosts de Azure Virtual Desktop mediante plantillas de Azure Resource Manager.

## Objetivos
  
Después de completar este laboratorio, podrá:

- Implementar grupos de hosts y hosts de Azure Virtual Desktop mediante plantillas de Azure Resource Manager

## Archivos de laboratorio

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## Instrucciones

### Ejercicio 1: Implementar grupos de hosts y hosts de Azure Virtual Desktop mediante plantillas de Azure Resource Manager
  
Las tareas principales de este ejercicio son las siguientes:

1. Preparación para la implementación de un grupo de hosts de Azure Virtual Desktop mediante una plantilla de Azure Resource Manager
1. Implementación de un grupo de hosts de Azure Virtual Desktop mediante una plantilla de Azure Resource Manager
1. Comprobación de la implementación del grupo de hosts y hosts de Azure Virtual Desktop
1. Preparación para la adición de un grupo de hosts de Azure Virtual Desktop existente mediante una plantilla de Azure Resource Manager
1. Adición de un grupo de hosts de Azure Virtual Desktop existente mediante una plantilla de Azure Resource Manager
1. Comprobación de los cambios en el grupo de hosts de Azure Virtual Desktop
1. Administración de asignaciones de escritorio personal en el grupo de hosts de Azure Virtual Desktop

#### Tarea 1: Preparación para la implementación de un grupo de hosts de Azure Virtual Desktop mediante una plantilla de Azure Resource Manager

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, haga clic en **az140-dc-vm11**.
1. En el panel **az140-dc-vm11**, seleccione **Conectar**, en el menú desplegable, seleccione **Conectar via Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Bastion a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para identificar el nombre distintivo de la unidad organizativa denominada **WVDInfra**, que hospedará los objetos de equipo del grupo de hosts de Azure Virtual Desktop:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para identificar el atributo del nombre principal de la cuenta de **Estudiante\\ADATUM**, que utilizará para unir los host de Azure Virtual Desktop con el dominio AD DS (**student@adatum.com**):

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para identificar el nombre principal de usuario de las cuentas **ADATUM\\aduser7** y **ADATUM\\aduser8** que utilizará para probar asignaciones personales más tarde en este laboratorio:

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **Nota**: Registre todos los valores de nombre principal de usuario que identificó **y** el nombre distintivo de la unidad organizativa WVDInfra. Los necesitará más adelante en este laboratorio.

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para calcular el tiempo de expiración del token necesario para realizar una implementación basada en una plantilla:

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **Nota**: El valor debería tener un formato similar a `2022-03-27T00:51:28.3008055Z`. Anótelo, ya que lo necesitará en la siguiente tarea.

   > **Nota**: Se necesita un token de registro para autorizar a un host a unirse al grupo. El valor de la fecha de expiración del token debe estar entre una hora y un mes a partir de la fecha y hora actuales.

1. En la sesión de Bastion a **az140-dc-vm11**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Si se le solicita, inicie sesión con las credenciales de Microsoft Entra de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Bastion a **az140-dc-vm11**, en Azure Portal, utilice el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a **Redes virtuales** y, en la hoja **Redes virtuales**, seleccione **az140-adds-vnet11**. 
1. En la hoja **az140-adds-vnet11**, seleccione **Subredes**, y en la hoja **Subredes**, seleccione **+ Subred**. Después, en la hoja **Agregar subred**, especifique la siguiente configuración (deje todas las demás opciones con sus valores predeterminados) y haga clic en **Guardar**:

   |Configuración|Value|
   |---|---|
   |Nombre|**hp2-Subnet**|
   |Intervalo de direcciones de subred|**10.0.2.0/24**|

1. En la sesión de Bastion para **az140-dc-vm11**, en Azure Portal, use el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a **Grupos de seguridad de red** y, en la hoja **Grupos de seguridad de red**, seleccione el único grupo de seguridad de red.
1. En la hoja Grupo de seguridad de red, en el menú vertical de la izquierda, en la sección **Configuración**, haga clic en **Propiedades**.
1. En la hoja **Propiedades**, haga clic en el icono **Copiar al Portapapeles** en el lado derecho del cuadro de texto **Id. de recurso**. 

   > **Nota**: El valor debe ser similar al formato `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, aunque el identificador de suscripción variará. Anótelo, ya que lo necesitará en la siguiente tarea.
1. Ahora debería haber registrado **seis** valores. Nombre distintivo, 3 nombres principales de usuario, un valor DateTime y el identificador de recurso. Si no tiene 6 valores registrados, vuelva a leer esta tarea **antes** de continuar. 

#### Tarea 2: Implementación de un grupo de hosts de Azure Virtual Desktop mediante una plantilla de Azure Resource Manager

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En el equipo de laboratorio, en la misma ventana del explorador web, abra otra pestaña del explorador web y vaya a la página del repositorio de plantillas de Azure RDS de GitHub [Plantilla de ARM para crear y aprovisionar un nuevo grupo de hosts de Azure Virtual Desktop](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool). 
1. En la página **Plantilla de ARM para crear y aprovisionar un nuevo grupo de hosts de Azure Virtual Desktop**, seleccione **Implementar en Azure**. Se redirigirá automáticamente el explorador a la hoja **Implementación personalizada** en Azure Portal.
1. En la hoja **Implementación personalizada**, seleccione **Editar parámetros**.
1. En la hoja **Editar parámetros**, seleccione **Cargar archivo**; en el cuadro de diálogo **Abrir**, seleccione **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json**, **Abrir** y, luego, **Guardar**. 
1. De vuelta en la hoja **Implementación personalizada**, especifique las opciones de configuración siguientes (deje las demás con los valores predeterminados):

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Grupo de recursos|cree un **nuevo** grupo de recursos denominado **az140-23-RG**|
   |Region|el nombre de la región de Azure en la que implementó máquinas virtuales de Azure que hospedan controladores de dominio de AD DS en el laboratorio **Preparación para la implementación de Azure Virtual Desktop (AD DS)**.|
   |Location|el nombre de la misma región de Azure que la establecida como el valor de los parámetros **Región**.|
   |Ubicación del área de trabajo|el nombre de la misma región de Azure que la establecida como el valor de los parámetros **Región**.|
   |Grupo de recursos del área de trabajo|none, ya que, si es NULL, su valor se establecerá automáticamente para que coincida con el grupo de recursos de destino de implementación.|
   |Referencia de todos los grupos de aplicaciones|ninguno, ya que no hay ningún grupo de aplicaciones existente en el área de trabajo de destino (no hay ningún área de trabajo).|
   |Ubicación de máquina virtual|el nombre de la misma región de Azure que la establecida como el valor de los parámetros **Ubicación**.|
   |Creación de un grupo de seguridad de red|**false**|
   |Identificador del grupo de seguridad de red|el valor del parámetro resourceID del grupo de seguridad de red existente que identificó en la tarea anterior.|
   |Hora de expiración del token| el valor de la hora de expiración del token que calculó en la tarea anterior.|

   > **Nota**: la implementación aprovisiona un grupo con el tipo de asignación de escritorio personal.

1. En la hoja **Implementación personalizada**, seleccione **Revisar y crear** y seleccione **Crear**.

   > **Nota**: Espere a que la implementación se complete antes de avanzar a la siguiente tarea. Esto puede tardar unos 15 minutos. 

#### Tarea 3: Comprobación de la implementación del grupo de hosts y hosts de Azure Virtual Desktop

1. Desde el equipo de laboratorio y, en el explorador web que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop**. En la hoja **Azure Virtual Desktop**, seleccione **Grupos de hosts** y, en la hoja **Grupos de hosts\| de Azure Virtual Desktop**, seleccione la entrada **az140-23-hp2** que representa el grupo recién implementado.
1. En la hoja **az140-23-hp2**, en el menú vertical del lado izquierdo, en la sección **Administrar**, haga clic en **Hosts de sesión**. 
1. En la hoja **Hosts de sesión de \|az140-23-hp2**, compruebe que la implementación consta de dos hosts.
1. En la hoja **Hosts de sesión de \|az140-23-hp2**, en el menú vertical del lado izquierdo, en la sección **Administrar**, haga clic en **Grupos de aplicaciones**.
1. En la hoja **Grupos de aplicaciones de \|az140-23-hp2**, compruebe que la implementación incluye el grupo de aplicaciones **Escritorio predeterminado** denominado **az140-23-hp2-DAG**.

#### Tarea 4: Preparación para la adición de un grupo de hosts de Azure Virtual Desktop existente mediante una plantilla de Azure Resource Manager

1. Desde el equipo de laboratorio, cambie la sesión de Bastion a **az140-dc-vm11**. 
1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para generar el token necesario para incorporar hosts nuevos al grupo que aprovisionó anteriormente en este ejercicio:

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para recuperar el valor del token y pegarlo en el Portapapeles:

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **Nota**: Registre el valor copiado en el Portapapeles (por ejemplo, iniciando el Bloc de notas y presionando la combinación de teclas Ctrl+V para pegar el contenido del Portapapeles en el Bloc de notas) cuyo contenido necesitará en la tarea siguiente. Asegúrese de que el valor que usa incluye una sola línea de texto, sin saltos de línea. 

   > **Nota**: Se necesita un token de registro para autorizar a un host a unirse al grupo. El valor de la fecha de expiración del token debe estar entre una hora y un mes a partir de la fecha y hora actuales.

#### Tarea 5: Adición de un grupo de hosts de Azure Virtual Desktop existente mediante una plantilla de Azure Resource Manager

1. En el equipo de laboratorio, en la misma ventana del explorador web, abra otra pestaña del explorador web y vaya a la página del repositorio de plantillas de Azure RDS de GitHub [Plantilla de ARM para agregar hosts de sesión a un grupo de hosts existente de Azure Virtual Desktop](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool). 
1. En la página **Plantilla de ARM para agregar hosts de sesión a un grupo de hosts existente de Azure Virtual Desktop**, seleccione **Implementar en Azure**. Se redirigirá automáticamente el explorador a la hoja **Implementación personalizada** en Azure Portal.
1. En la hoja **Implementación personalizada**, seleccione **Editar parámetros**.
1. En la hoja **Editar parámetros**, seleccione **Cargar archivo**; en el cuadro de diálogo **Abrir**, seleccione **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json**, **Abrir** y, luego, **Guardar**. 
1. De vuelta en la hoja **Implementación personalizada**, especifique las opciones de configuración siguientes (deje las demás con los valores predeterminados):

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Grupo de recursos|**az140-23-RG**|
   |Token de grupo de hosts|el valor del token que generó en la tarea anterior.|
   |Ubicación del grupo de hosts|el nombre de la región de Azure en la que implementó el grupo de hosts anteriormente en este laboratorio.|
   |Ubicación de máquina virtual|el nombre de la misma región de Azure que la establecida como el valor de los parámetros **Ubicación de grupo de hosts**.|
   |Creación de un grupo de seguridad de red|**false**|
   |Identificador del grupo de seguridad de red|el valor del parámetro resourceID del grupo de seguridad de red existente que identificó en la tarea anterior.|

1. En la hoja **Implementación personalizada**, seleccione **Revisar y crear** y seleccione **Crear**.

   > **Nota**: Espere a que la implementación se complete antes de avanzar a la siguiente tarea. Esto puede tardar unos 10 minutos.

#### Tarea 6: Comprobación de los cambios en el grupo de hosts de Azure Virtual Desktop

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, tenga en cuenta que la lista incluye una máquina virtual adicional denominada **az140-23-p2-2**.
1. Desde el equipo de laboratorio, cambie la sesión de Bastion a **az140-dc-vm11**. 
1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para verificar que el tercer host se unió correctamente al dominio AD DS **adatum.com**:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. Vuelva al equipo de laboratorio y, en el explorador web que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop**. En la hoja **Azure Virtual Desktop**, seleccione **Grupos de hosts** y, en la hoja **Grupos de hosts\| de Azure Virtual Desktop**, seleccione la entrada **az140-23-hp2** que representa el grupo recién modificado.
1. En la hoja **az140-23-hp2**, revise la sección **Essentials** y compruebe que el **Tipo de grupo de hosts** está establecido en **Personal** con el **Tipo de asignación** establecido en **Automática**.
1. En la hoja **az140-23-hp2**, en el menú vertical del lado izquierdo, en la sección **Administrar**, haga clic en **Hosts de sesión**. 
1. En la hoja **Hosts de sesión de \|az140-23-hp2**, compruebe que la implementación consta de tres hosts. 

#### Tarea 7: Administración de asignaciones de escritorio personal en el grupo de hosts de Azure Virtual Desktop

1. En el equipo de laboratorio y, en el explorador web que muestra Azure Portal, en la hoja **Hosts de sesión de \|az140-23-hp2**, en el menú vertical del lado izquierdo, en la sección **Administrar**, seleccione **Grupos de aplicaciones**. 
1. En la hoja **Grupos de aplicaciones de \|az140-23-hp2**, en la lista de grupos de aplicaciones, seleccione **az140-23-hp2-DAG**.
1. En la hoja **az140-23-hp2-DAG**, en el menú vertical de la izquierda, seleccione **Asignaciones**. 
1. En la hoja **Asignaciones de \|az140-23-hp2-DAG**, seleccione **+ Agregar**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, seleccione **Grupos**, y a continuación, seleccione **az140-wvd-personal** y haga clic en **Seleccionar**.

   > **Nota**: Ahora vamos a revisar la experiencia de un usuario que se conecta al grupo de hosts de Azure Virtual Desktop.

1. Desde el equipo de laboratorio, en la ventana del explorador donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11**.
1. En la hoja **az140-cl-vm11**, seleccione **Conectar**, en el menú desplegable, seleccione**Conectar a través de Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Bastion a **az140-cl-vm11**, haga clic en **Inicio** y, en el menú **Inicio**, seleccione la aplicación del cliente **Escritorio remoto**.
2. En la ventana Escritorio remoto, haga clic en el icono de puntos suspensivos de la esquina superior derecha, en el menú desplegable, haga clic en **Cancelar suscripción** y, cuando se le solicite confirmación, haga clic en **Continuar**.
3. Dentro de la sesión de Bastion a **az140-cl-vm11**, en la ventana Escritorio remoto, en la página **Empecemos**, haga clic en **Suscribirse**.
4. En la ventana de cliente de **Escritorio remoto**, seleccione **Suscribirse** y, cuando se le solicite, inicie sesión con las credenciales de **aduser7**: proporcione el atributo userPrincipalName y la contraseña que estableció al crear la cuenta de usuario.

   > **Nota**: Como alternativa, en la ventana del cliente **Escritorio remoto**, seleccione **Suscribirse con dirección URL**, y, en el panel **Suscribirse a un área de trabajo**, en **URL del correo electrónico o área de trabajo**, escriba **https://client.wvd.microsoft.com/api/arm/feeddiscovery**, seleccione **Siguiente** y, una vez que se le solicite, inicie sesión con las credenciales de **aduser7** (con el atributo userPrincipalName como el nombre de usuario y la contraseña que estableció al crear esta cuenta). 

1. En la página **Escritorio remoto**, haga doble clic en el icono **SessionDesktop** y, cuando se le soliciten las credenciales, vuelva a escribir la misma contraseña, active la casilla **Recordarme** y haga clic en **Aceptar**.
1. Compruebe que **aduser7** ha iniciado sesión correctamente mediante Escritorio remoto en un host.
1. Dentro de la sesión de Escritorio remoto a uno de los hosts como **aduser7**, haga clic con el botón derecho en **Inicio**, y, en el menú contextual, seleccione **Apagar o cerrar sesión** y, en el menú en cascada, haga clic en **Cerrar sesión**.

   > **Nota**: Ahora vamos a cambiar la asignación de escritorio personal del modo directo al automático. 

1. Cambie al equipo de laboratorio, al explorador web que muestra el Azure Portal, y en la hoja **Asignaciones de \|az140-23-hp2-DAG**, en la barra informativa situada directamente encima de la lista de asignaciones, haga clic en el vínculo **Asignar máquina virtual**. Esto le redirigirá a la hoja **Hosts de sesión de \|az140-23-hp2**. 
1. En la hoja **Hosts de sesión de \|az140-23-hp2**, compruebe que uno de los hosts tiene **aduser7** enumerado en la columna **Usuario asignado**.

   > **Nota**: Esto se espera, ya que el grupo de hosts está configurado para la asignación automática.

1. En el equipo de laboratorio, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel Cloud Shell, ejecute lo siguiente para cambiar al modo de asignación directa:

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, vaya a la hoja del grupo de hosts **az140-23-hp2**, revise la sección **Essentials** y compruebe que el **Tipo de grupo de hosts** está establecido en **Personal**, con el **Tipo de asignación** establecido en **Directa**.
1. Vuelva a la sesión de Bastion a **az140-cl-vm11**, en la ventana de **Escritorio remoto**, y haga clic en el icono de puntos suspensivos en la esquina superior derecha, en el menú contextual, y haga clic en **Cancelar Suscripción**; cuando se le solicite confirmación, haga clic en **Continuar**.
1. Dentro de la sesión de Bastion a **az140-cl-vm11**, en la ventana de **Escritorio remoto**, en la página **Empecemos**, haga clic en **Suscribirse**.
1. Cuando se le pida que inicie sesión, en el panel **Elegir una cuenta**, haga clic en **Usar otra cuenta** y, cuando se le solicite, inicie sesión con el nombre principal de usuario de la cuenta de usuario **aduser8** con la contraseña que estableció al crear esta cuenta.
1. En la página **Escritorio remoto**, haga doble clic en el icono **SessionDesktop** y compruebe que recibe el mensaje de error **No se ha podido establecer la conexión porque actualmente no hay recursos disponibles. Vuelva a conectarse más tarde o, si sigue ocurriendo, póngase en contacto con el soporte técnico para obtener ayuda**, y haga clic en **Aceptar**.

   > **Nota**: Esto se espera, ya que el grupo de hosts está configurado para la asignación directa y no se ha asignado un host a **aduser8**.

1. Cambie al equipo de laboratorio, al explorador web que muestra Azure Portal y, en la hoja **Hosts de sesión de \|az140-23-hp2**, seleccione el vínculo **(Asignar)** de la columna **Usuario asignado** junto a uno de los dos hosts sin asignar restantes.
1. En **Asignar un usuario**, seleccione **aduser8**, haga clic en **Seleccionar** y, cuando se le pida confirmación, haga clic en **Aceptar**.
1. Vuelva a la sesión de Bastion a **az140-cl-vm11**, en la ventana **Escritorio remoto**, haga doble clic en el icono **SessionDesktop** y, cuando se le solicite la contraseña, escriba la contraseña establecida al crear esta cuenta de usuario, haga clic en **Aceptar** y compruebe que puede iniciar sesión correctamente en el host asignado.
1. En el escritorio de sesión del host asignado para **aduser8**, haga clic con el botón derecho en **Inicio**, en el menú contextual, seleccione **Apagar o cerrar sesión** y, en el menú en cascada, haga clic en **Cerrar sesión**.
1. En la sesión de Bastion para **az140-cl-vm11**, haga clic con el botón derecho en **Inicio**, en el menú contextual, seleccione **Apagar o cerrar sesión** y, en el menú en cascada, haga clic en **Cerrar**, y a continuación, haga clic en **Cerrar**.

### Ejercicio 2: Detener y desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detención y desasignación de máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignará las máquinas virtuales de Azure aprovisionadas en este laboratorio para minimizar los cargos de proceso correspondientes.

#### Tarea 1: Desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para enumerar todas las máquinas virtuales de Azure creadas en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. Desde la sesión de PowerShell, en el panel Cloud Shell, ejecute lo siguiente para detener y desasignar todas las máquinas virtuales de Azure que creó en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.
