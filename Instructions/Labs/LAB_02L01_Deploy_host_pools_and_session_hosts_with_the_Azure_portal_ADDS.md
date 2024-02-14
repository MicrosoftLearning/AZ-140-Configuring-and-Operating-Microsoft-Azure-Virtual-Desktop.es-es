---
lab:
  title: "Laboratorio: Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (AD\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratorio: Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (AD DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure que usará en este laboratorio.
- Una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario o Colaborador en la suscripción de Azure que va a usar en este laboratorio y con el rol Administrador global en el inquilino de Microsoft Entra asociado a esa suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**

## Tiempo estimado

60 minutos

## Escenario del laboratorio

Debe crear y configurar grupos de hosts y hosts de sesión en un entorno de Active Directory Domain Services (AD DS).

## Objetivos
  
Después de completar este laboratorio, podrá:

- Implementar un entorno de Azure Virtual Desktop en un dominio de Azure AD DS.
- Validar un entorno de Azure Virtual Desktop en un dominio de Azure AD DS.

## Archivos de laboratorio

- None

## Instrucciones

### Ejercicio 1: Implementar un entorno de Azure Virtual Desktop en un dominio de Azure AD DS.
  
Las tareas principales de este ejercicio son las siguientes:

1. Preparación del dominio de AD DS y la suscripción de Azure para la implementación de un grupo de hosts de Azure Virtual Desktop
1. Implementación de un grupo de hosts de Azure Virtual Desktop
1. Administración de los hosts de sesión del grupo de hosts de Azure Virtual Desktop
1. Configuración de grupos de aplicaciones de Azure Virtual Desktop
1. Configuración de áreas de trabajo de Azure Virtual Desktop

#### Tarea 1: Preparación del dominio de AD DS y la suscripción de Azure para la implementación de un grupo de hosts de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal]( ) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, haga clic en **az140-dc-vm11**.
1. En la hoja **az140-dc-vm11**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-dc-vm11 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear una unidad organizativa que hospedará los objetos de equipo de los hosts de Azure Virtual Desktop:

   ```powershell
   New-ADOrganizationalUnit 'WVDInfra' –path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para iniciar sesión en la suscripción de Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Cuando se le solicite, proporcione las credenciales de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para identificar el nombre de usuario principal de la cuenta **aduser1**:

   ```powershell
   (Get-AzADUser -DisplayName 'aduser1').UserPrincipalName
   ```

   > **Nota**: Registre el nombre principal de usuario que identificó en este paso. Lo necesitará más adelante en este laboratorio.

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para registrar el proveedor de recursos de **Microsoft.DesktopVirtualization**:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Si se le solicita, inicie sesión con las credenciales de Microsoft Entra de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en Azure Portal, utilice el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a **Redes virtuales** y, en la hoja **Redes virtuales**, seleccione **az140-adds-vnet11**. 
1. En la hoja **az140-adds-vnet11**, seleccione **Subredes**, y en la hoja **Subredes**, seleccione **+ Subred**. Después, en la hoja **Agregar subred**, especifique la siguiente configuración (deje todas las demás opciones con sus valores predeterminados) y haga clic en **Guardar**:

   |Configuración|Value|
   |---|---|
   |Nombre|**hp1-Subnet**|
   |Intervalo de direcciones de subred|**10.0.1.0/24**|

#### Tarea 2: Implementación de un grupo de hosts de Azure Virtual Desktop

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop**; en la hoja **Azure Virtual Desktop**, seleccione **Grupos de hosts** y, en la hoja **Grupos de hosts de \|Azure Virtual Desktop**, seleccione **+Crear**. 
1. En la pestaña **Aspectos básicos** de la hoja **Crear un grupo de hosts**, especifique la siguiente configuración y seleccione **Siguiente: Máquinas virtuales >** (deje el resto de la configuración con los valores predeterminados)

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|nombre de un nuevo grupo de recursos **az140-21-RG**.|
   |Nombre del grupo de hosts|**az140-21-hp1**|
   |Location|nombre de la región de Azure en la que implementó recursos en el primer ejercicio de este laboratorio o región cercana a la misma |
   |Entorno de validación|**No**|
   |Tipo de grupo de aplicaciones preferido|**Dispositivo de escritorio**|
   |Tipo de grupo de hosts|**Agrupado**|
   |Algoritmo de equilibrio de carga|**Con prioridad a la amplitud**|
   |Límite máximo de sesión|**12**
          |

1. En la pestaña **Máquinas virtuales** de la hoja **Crear un grupo de hosts**, especifique la siguiente configuración y seleccione **Siguiente: Área de trabajo >** (deje las opciones restantes con sus valores predeterminados)

   |Configuración|Valor|
   |---|---|
   |Agregar Máquinas virtuales de Azure|**Sí**|
   |Resource group|**El valor predeterminado es el mismo que el del grupo de hosts**|
   |Prefijo de nombre|**az140-21-p1**|
   |Ubicación de la máquina virtual|nombre de la región de Azure en la que implementó recursos en el primer ejercicio de este laboratorio|
   |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
   |Tipo de seguridad|**Estándar**|
   |Imagen|**Windows 11 Enterprise para sesiones múltiples y Aplicaciones de Microsoft 365, versión 22H2**|
   |Tamaño de la máquina virtual|**Estándar D2s v3**|
   |Número de VM|**2**|
   |Tipo de disco del sistema operativo|**SSD estándar**|
   |Diagnósticos de arranque|**Habilitar con la cuenta de almacenamiento administrada (recomendado)**|
   |Red virtual|**az140-adds-vnet11**|
   |Subnet|**hp1-Subnet (10.0.1.0/24)**|
   |Grupo de seguridad de red|**Basic**|
   |Puertos de entrada públicos|**No**|
   |Seleccione el directorio al que quiera unirse|**Active Directory**|
   |UPN de unión a un dominio de AD|**student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|
   |Especifique el nombre de dominio o la unidad|**Sí**|
   |Dominio al que desea unirse|**adatum.com**|
   |Ruta de acceso de unidad organizativa|**OU=WVDInfra,DC=adatum,DC=com**|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|
   |Confirmar contraseña|**Pa55w.rd1234**|

1. En la pestaña **Área de trabajo** de la hoja **Crear un grupo de hosts**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicación de escritorio|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear un grupo de hosts**, seleccione **Crear**.

   > **Nota**: Espere a que la implementación se complete. Esto puede tardar unos 10 minutos.

#### Tarea 3: Administración de los hosts de sesión del grupo de hosts de Azure Virtual Desktop

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana del explorador web que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, en la barra de menús vertical, en la sección **Administrar**, seleccione **Grupos de hosts**.
1. En la hoja **Grupos de hosts de \|Azure Virtual Desktop**, en la lista de grupos de hosts, seleccione **az140-21-hp1**.
1. En la hoja **az140-21-hp1**, en la barra de menús vertical, en la sección **Administrar**, seleccione **Hosts de sesión** y compruebe que el grupo consta de dos hosts. 
1. En la hoja **Hosts de sesión de \|az140-21-hp1**, seleccione **+Agregar**.
1. En la pestaña **Aspectos básicos** de la hoja **Agregar máquinas virtuales al grupo de hosts**, revise los ajustes preconfigurados y seleccione **Siguiente: Máquinas virtuales**.
1. En la pestaña **Máquinas virtuales** de la hoja **Agregar máquinas virtuales a un grupo de hosts**, especifique la siguiente configuración y seleccione **Revisar y crear** (deje los otros valores con su configuración predeterminada):

   |Configuración|Value|
   |---|---|
   |Resource group|**az140-21-RG**|
   |Prefijo de nombre|**az140-21-p1**|
   |Ubicación de la máquina virtual|el nombre de la región de Azure en la que implementó las dos primeras máquinas virtuales del host de sesión.|
   |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
   |Tipo de seguridad|**Estándar**|
   |Imagen|**Windows 11 Enterprise para sesiones múltiples y Aplicaciones de Microsoft 365, versión 22H2**|
   |Número de VM|**1**|
   |Tipo de disco del sistema operativo|**SSD estándar**|
   |Diagnósticos de arranque|**Habilitar con la cuenta de almacenamiento administrada (recomendado)**|
   |Red virtual|**az140-adds-vnet11**|
   |Subnet|**hp1-Subnet (10.0.1.0/24)**|
   |Grupo de seguridad de red|**Basic**|
   |Puertos de entrada públicos|**No**|
   |UPN de unión a un dominio de AD|**student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|
   |Especifique el nombre de dominio o la unidad|**Sí**|
   |Dominio al que desea unirse|**adatum.com**|
   |Ruta de acceso de unidad organizativa|**OU=WVDInfra,DC=adatum,DC=com**|   
   |Nombre de usuario de la cuenta de administrador de máquina virtual|**Estudiante**|
   |Contraseña de la cuenta de administrador de máquina virtual|**Pa55w.rd1234**|

   > **Nota**: Como es probable que haya observado, es posible cambiar la imagen y el prefijo de las máquinas virtuales a medida que agrega hosts de sesión al grupo existente. En general, esto no se recomienda a menos que planee reemplazar todas las máquinas virtuales del grupo. 

1. En la pestaña **Revisar y crear** de la hoja **Agregar máquinas virtuales a un grupo de hosts**, seleccione **Crear**.

   > **Nota**: Espere a que la implementación se complete antes de avanzar a la siguiente tarea. La implementación puede tardar unos 5 minutos. 

#### Tarea 4: Configuración de grupos de aplicaciones de Azure Virtual Desktop

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana del explorador web que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, seleccione **Grupos de aplicaciones**.
1. En la hoja **Grupos de aplicaciones de \|Azure Virtual Desktop**, busque el grupo de aplicaciones de escritorio existente y generado automáticamente **az140-21-hp1-DAG** y selecciónelo. 
1. En la hoja **az140-21-hp1-DAG**, seleccione **Asignaciones**.
1. En la hoja **Asignaciones de \|az140-21-hp1-DAG**, seleccione **+ Agregar**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, seleccione **az140-wvd-pooled** y haga clic en **Seleccionar**.
1. Vuelva a la hoja **Grupos de aplicaciones de \|Azure Virtual Desktop** y seleccione **+ Crear**. 
1. En la pestaña **Aspectos básicos** de la hoja **Crear un grupo de aplicación**, especifique la siguiente configuración y seleccione **Siguiente: Aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az140-21-RG**|
   |Grupo de hosts|**az140-21-hp1**|
   |Tipo de grupo de aplicaciones|**Aplicación remota (RAIL)**|
   |Nombre del grupo de aplicaciones|**az140-21-hp1-Office365-RAG**|

1. En la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicaciones**, especifique la configuración siguiente y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Menú Inicio**|
   |Aplicación|**Word**|
   |Descripción|**Microsoft Word**|
   |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicaciones**, especifique la configuración siguiente y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Menú Inicio**|
   |Aplicación|**Excel**|
   |Descripción|**Microsoft Excel**|
   |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicaciones**, especifique la configuración siguiente y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Menú Inicio**|
   |Aplicación|**PowerPoint**|
   |Descripción|**Microsoft PowerPoint**|
   |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **Siguiente: Asignaciones >**.
1. En la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, seleccione **az140-wvd-remote-app** y haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **Siguiente: Workspace >** (Siguiente: Área de trabajo).
1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear grupo de aplicaciones**, seleccione **Crear**.

   > **Nota**: Espere a que se cree el grupo de aplicaciones. Debería tardar menos de un minuto. 

   > **Nota**: Ahora creará un grupo de aplicaciones basado en la ruta de acceso del archivo como origen de la aplicación.

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, seleccione **Grupos de aplicaciones**.
1. En la hoja **Grupos de aplicaciones \|de Azure Virtual Desktop**, seleccione **+ Crear**. 
1. En la pestaña **Aspectos básicos** de la hoja **Crear un grupo de aplicación**, especifique la siguiente configuración y seleccione **Siguiente: Aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az140-21-RG**|
   |Grupo de hosts|**az140-21-hp1**|
   |Tipo de grupo de aplicaciones|**RemoteApp (RAIL)**|
   |Nombre del grupo de aplicaciones|**az140-21-hp1-Utilities-RAG**|

1. En la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicación**, use las pestañas **Básico** e **Índice** para especificar la siguiente configuración y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Ruta de acceso del archivo**|
   |Application path|**C:\Windows\system32\cmd.exe**|
   |Nombre de la aplicación|**Símbolo del sistema**|
   |Nombre para mostrar|**Símbolo del sistema**|
   |Ruta de icono|**C:\Windows\system32\cmd.exe**|
   |Índice de icono|**0**|
   |Descripción|**Símbolo del sistema de Windows**|
   |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **Siguiente: Asignaciones >**.
1. En la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, seleccione **az140-wvd-remote-app** y **az140-wvd-admins** y haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **Siguiente: Workspace >** (Siguiente: Área de trabajo).
1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear grupo de aplicaciones**, seleccione **Crear**.

#### Tarea 5: Configuración de áreas de trabajo de Azure Virtual Desktop

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, seleccione **Áreas de trabajo**.
1. En la hoja **Áreas de trabajo de \|Azure Virtual Desktop**, seleccione **+ Crear**. 
1. En la pestaña **Aspectos básicos** de la hoja **Crear un área de trabajo**, especifique la siguiente configuración y seleccione **Siguiente: Grupos de aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az140-21-RG**|
   |Nombre del área de trabajo|**az140-21-ws1**|
   |Nombre descriptivo|**az140-21-ws1**|
   |Location|nombre de la región de Azure en la que implementó recursos en el primer ejercicio de este laboratorio o región cercana a la misma|

1. En la pestaña **Grupos de aplicaciones** de la hoja **Crear un área de trabajo**, especifique la siguiente configuración:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**Sí**|

1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, seleccione **+ Registrar grupos de aplicaciones**.
1. En la hoja **Agregar grupos de aplicaciones**, seleccione el signo más junto a las entradas **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** y **az140-21-hp1-Utilities-RAG** y haga clic en **Seleccionar**. 
1. De nuevo en la pestaña **Grupos de aplicaciones** de la hoja **Crear un área de trabajo**, seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear** de la hoja **Crear un área de trabajo**, seleccione **Crear**.

### Ejercicio 2: Validación de un entorno de Azure Virtual Desktop
  
Las tareas principales de este ejercicio son las siguientes:

1. Instalación del cliente Escritorio remoto de Microsoft (MSRDC) en un equipo con Windows 10
1. Suscripción a un área de trabajo de Azure Virtual Desktop
1. Prueba de aplicaciones de Azure Virtual Desktop

#### Tarea 1: Instalación del cliente Escritorio remoto de Microsoft (MSRDC) en un equipo con Windows 10

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana del explorador que muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11**.
1. En la hoja **az140-cl-vm11**, desplácese hacia abajo hasta la sección **Operaciones** y seleccione **Ejecutar comando**. 
1. En la hoja **Ejecutar comandos de \|az140-cl-vm11**, seleccione **EnableRemotePS** y seleccione **Ejecutar**. 

   > **Nota**: Espere a que el comando se complete antes de avanzar al siguiente paso. Esto puede tardar 1 minuto. Puede obtener errores de texto rojo en relación al perfil público que se usa y no el perfil de dominio; si es así, puede omitirlos e ir al paso siguiente.

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde la consola de **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para agregar todos los miembros de **ADATUM\\az140-wvd-users** al grupo local **Usuarios de Escritorio remoto** en la máquina virtual Azure **az140-cl-vm11** que ejecuta Windows 10 que implementó en el laboratorio **Preparación de la implementación de Azure Virtual Desktop (AD DS)**.

   ```powershell
   $computerName = 'az140-cl-vm11'
   Invoke-Command -ComputerName $computerName -ScriptBlock {Add-LocalGroupMember -Group 'Remote Desktop Users' -Member 'ADATUM\az140-wvd-users'}
   ```

1. Cambie al equipo de laboratorio y desde él, en la ventana del explorador donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11**.
1. En la hoja **az140-cl-vm11**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-cl-vm11 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-cl-vm11**, inicie Microsoft Edge y vaya a la [página de descarga del cliente de escritorio de Windows](https://go.microsoft.com/fwlink/?linkid=2068602) y, cuando se le solicite, seleccione **Ejecutar** para iniciar su instalación. En la página **Ámbito de instalación** del asistente de **Instalación de Escritorio remoto**, seleccione la opción **Instalar para todos los usuarios de esta máquina** y haga clic en **Instalar**. Si el Control de cuentas de usuario le solicita las credenciales administrativas, autentíquese mediante el nombre de usuario de **ADATUM\\Student** con **Pa55w.rd1234** como contraseña.
1. Una vez completada la instalación, asegúrese de que la casilla **Iniciar Escritorio remoto cuando se cierre la instalación** esté activada y haga clic en **Finalizar** para iniciar el cliente de Escritorio remoto.

#### Tarea 2: Suscripción a un área de trabajo de Azure Virtual Desktop

1. En la ventana de cliente de **Escritorio remoto**, seleccione **Suscribirse** y, cuando se le solicite, inicie sesión con las credenciales de **aduser1**: proporcione el atributo userPrincipalName que identificó anteriormente ne este laboratorio y la contraseña que estableció al crear la cuenta.

   > **Nota**: Como alternativa, en la ventana del cliente **Escritorio remoto**, seleccione **Suscribirse con dirección URL**, y, en el panel **Suscribirse a un área de trabajo**, en **URL del correo electrónico o área de trabajo**, escriba **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, seleccione **Siguiente** y, una vez que se le solicite, inicie sesión con las credenciales de **aduser1** (con el atributo userPrincipalName como el nombre de usuario y la contraseña que estableció al crear esta cuenta). 

1. Si se solicita, en la ventana **Mantener sesión iniciada en todas las aplicaciones**, desactive la casilla **Permitir que mi organización administre el dispositivo** y seleccione **No, iniciar sesión solo en esta aplicación**. 
1. Asegúrese de que la página **Escritorio remoto** muestra la lista de aplicaciones que se incluyen en los grupos de aplicaciones asociados al área de trabajo y asociados con la cuenta de usuario **aduser1** mediante su pertenencia a grupos. 

#### Tarea 3: Prueba de aplicaciones de Azure Virtual Desktop

1. En la sesión de Escritorio remoto a **az140-cl-vm11**, en la ventana de cliente de **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **Símbolo del sistema** y compruebe que inicia una ventana de **Símbolo del sistema**. Cuando se le pida que se autentique, escriba la contraseña que estableció al crear la cuenta de usuario **aduser1**, active la casilla **Recordarme** y seleccione **Aceptar**.

   > **Nota**: Inicialmente, la aplicación puede tardar unos minutos en iniciarse pero, posteriormente, el inicio de la aplicación debe ser mucho más rápido.

   > **Nota**: Si aparece el mensaje de solicitud de inicio de sesión **Le damos la bienvenida a Microsoft Teams**, ciérrelo.

1. En el Símbolo del sistema, escriba **hostname** y presione la tecla **Intro** para mostrar el nombre del equipo en el que se ejecuta el Símbolo del sistema.

   > **Nota**: Compruebe que el nombre mostrado sea **az140-21-p1-0**, **az140-21-p1-1** o **az140-21-p1-2**, en lugar de **az140-cl-vm11**.

1. En el Símbolo del sistema, escriba **logoff** y presione la tecla **Intro** para cerrar sesión desde la sesión de aplicación remota actual.
1. En la sesión de Escritorio remoto a **az140-cl-vm11**, en la ventana del cliente **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **SessionDesktop** y compruebe que inicia sesión en Escritorio remoto. 
1. En la sesión **Escritorio predeterminado**, haga clic con el botón derecho en **Inicio**, seleccione **Ejecutar**, en el cuadro de texto **Abrir** del cuadro de diálogo **Ejecutar**, escriba **cmd** y seleccione **Aceptar**. 
1. En la sesión **Escritorio predeterminado**, en el símbolo del sistema, escriba **hostname** y presione la tecla **Intro** para mostrar el nombre del equipo en el que se ejecuta la sesión de Escritorio remoto.
1. Compruebe que el nombre mostrado sea **az140-21-p1-0**, **az140-21-p1-1** o **az140-21-p1-2**.

### Ejercicio 3: Detener y desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detención y desasignación de máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignará las máquinas virtuales de Azure aprovisionadas en este laboratorio para minimizar los cargos de proceso correspondientes.

#### Tarea 1: Desasignar máquinas virtuales de Azure aprovisionadas en el laboratorio

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
