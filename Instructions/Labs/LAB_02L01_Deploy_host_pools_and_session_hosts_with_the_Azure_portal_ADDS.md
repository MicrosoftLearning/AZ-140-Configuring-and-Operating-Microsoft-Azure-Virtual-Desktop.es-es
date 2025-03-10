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

- Ninguno

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

1. En la sesión de Bastion a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para crear una unidad organizativa que hospedará los objetos de equipo de los hosts de Azure Virtual Desktop:

   ```powershell
   New-ADOrganizationalUnit 'WVDInfra' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para identificar el nombre de usuario principal de la cuenta **aduser1**:

   ```powershell
   (Get-AzADUser -DisplayName 'aduser1').UserPrincipalName
   ```

   > **Nota**: Registre el nombre principal de usuario que identificó en este paso. Lo necesitará más adelante en este laboratorio.

1. Desde la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para registrar el proveedor de recursos de **Microsoft.DesktopVirtualization**:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. En la sesión de Bastion a **az140-dc-vm11**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Si se le solicita, inicie sesión con las credenciales de Microsoft Entra de la cuenta de usuario con el rol Propietario en la suscripción que usa en este laboratorio.
1. En la sesión de Bastion a **az140-dc-vm11**, en Azure Portal, utilice el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a **Redes virtuales** y, en la hoja **Redes virtuales**, seleccione **az140-adds-vnet11**. 
1. En la hoja **az140-adds-vnet11**, seleccione **Subredes**, y en la hoja **Subredes**, seleccione **+ Subred**. Después, en la hoja **Agregar subred**, especifique la siguiente configuración (deje todas las demás opciones con sus valores predeterminados) y haga clic en **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Nombre|**hp1-Subnet**|
   |Dirección inicial|**10.0.1.0**|
   |Size|**/24 (256 direcciones)**|

#### Tarea 2: Implementación de un grupo de hosts de Azure Virtual Desktop

1. Dentro de la sesión de Bastion a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra el Azure Portal, busca y selecciona **Azure Virtual Desktop**, en el panel **Azure Virtual Desktop**, selecciona **Grupos de host**, y en la hoja **Azure Virtual Desktop \| Grupos de host**, selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la hoja **Crear un grupo de hosts**, especifica la siguiente configuración y selecciona **Siguiente: Máquinas virtuales >** (deja los otros valores con su configuración predeterminada):

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|crear un **nuevo** grupo de recursos denominado **az140-21-RG**|
   |Nombre del grupo de hosts|**az140-21-hp1**|
   |Location|nombre de la región de Azure en la que implementó recursos en el primer ejercicio de este laboratorio o región cercana a la misma |
   |Entorno de validación|**No**|
   |Tipo de grupo de aplicaciones preferido|**Dispositivo de escritorio**|
   |Tipo de grupo de hosts|**Agrupado**|
   |Algoritmo de equilibrio de carga|**Equilibrio de carga en amplitud**|
   |Límite máximo de sesión|**12**
          |

   > **Nota**: Si un usuario tiene aplicaciones de Escritorio y RemoteApp publicadas, el tipo de grupo de aplicaciones preferido determina cuál de ellas aparecerá en su fuente.

1. En la pestaña **Máquinas virtuales** de la hoja **Crear un grupo de hosts**, especifique la siguiente configuración y seleccione **Siguiente: Área de trabajo >** (deja los otros valores con su configuración predeterminada):

   |Configuración|Valor|
   |---|---|
   |Agregar máquinas virtuales|**Sí**|
   |Grupo de recursos|**El valor predeterminado es el mismo que el del grupo de hosts**|
   |Prefijo de nombre|**az140-21-p1**|
   |Tipo de máquina virtual|Máquina virtual de Azure|
   |Ubicación de la máquina virtual|el nombre de la región Azure en la que implementó los recursos en el laboratorio anterior|
   |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
   |Tipo de seguridad|**Máquina virtual de inicio seguro**|
   |Imagen|**Windows 11 Enterprise para sesiones múltiples y Aplicaciones de Microsoft 365, versión 22H2**|
   |Tamaño de la máquina virtual|**Standard DC2s_v3**|
   |Número de máquinas virtuales|**2**|
   |Tipo de disco del sistema operativo|**SSD estándar**|
   |Tamaño de disco del SO|**Tamaño predeterminado (128GiB)**|
   |Diagnósticos de arranque|**Habilitar con la cuenta de almacenamiento administrada (recomendado)**|
   |Red virtual|**az140-adds-vnet11**|
   |Subred|**hp1-Subnet (10.0.1.0/24)**|
   |Grupo de seguridad de red|**Basic**|
   |Puertos de entrada públicos|**No**|
   |Seleccionar el directorio al que quieras unirte|**Active Directory**|
   |UPN de unión a un dominio de AD|**student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|
   |Confirmar contraseña|**Pa55w.rd1234**|
   |Especificar el nombre de dominio o la unidad|**Sí**|
   |Dominio al que deseas unirte|**adatum.com**|
   |Ruta de acceso de unidad organizativa|**OU=WVDInfra,DC=adatum,DC=com**|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|
   |Confirmar contraseña|**Pa55w.rd1234**|

1. En la pestaña **Área de trabajo** del panel **Crear un grupo de hosts**, confirme la siguiente configuración y seleccione **Revisar + crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones de escritorio|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear un grupo de hosts**, selecciona **Crear**.

   > **Nota**: espera a que la implementación se complete. Esto puede llevar unos 10-15 minutos.

#### Tarea 3: Administración de los hosts de sesión del grupo de hosts de Azure Virtual Desktop

1. Dentro de la sesión Bastion a **az140-dc-vm11**, en la ventana del navegador web que muestra el portal Azure, busca y selecciona **Azure Virtual Desktop** y, en el panel **Azure Virtual Desktop**, en la barra de menú vertical, en la **sección Administrar**, selecciona **Grupos de hosts**.
1. En la hoja **Grupos de hosts de \|Azure Virtual Desktop**, en la lista de grupos de hosts, selecciona **az140-21-hp1**.
1. En la hoja **az140-21-hp1**, en la barra de menús vertical, en la sección **Administrar**, selecciona **Hosts de sesión** y comprueba que el grupo consta de dos hosts. 
1. En la hoja **az140-21-hp1 \| Hosts de sesión**, selecciona **+ Agregar**.
1. En la pestaña **Datos básicos** de la hoja **Agregar máquinas virtuales al grupo de hosts**, revisa la configuración preconfigurada y selecciona **Siguiente: Máquinas virtuales**.
1. En la pestaña **Máquinas virtuales** de la hoja **Agregar máquinas virtuales a un grupo de hosts**, especifica la siguiente configuración y selecciona **Revisar y crear** (deja los otros valores con su configuración predeterminada):

   |Configuración|Valor|
   |---|---|
   |Resource group|**az140-21-RG**|
   |Prefijo de nombre|**az140-21-p1**|
   |Ubicación de la máquina virtual|el nombre de la región de Azure en la que implementó las dos primeras máquinas virtuales del host de sesión.|
   |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
   |Tipo de seguridad|**Máquina virtual de inicio seguro**|
   |Imagen|**Windows 11 Enterprise para sesiones múltiples y Aplicaciones de Microsoft 365, versión 22H2**|
   |Número de VM|**1**|
   |Tipo de disco del sistema operativo|**SSD estándar**|
   |Tamaño de disco del SO|**Tamaño predeterminado (128GiB)**|
   |Diagnósticos de arranque|**Habilitar con la cuenta de almacenamiento administrada (recomendado)**|
   |Red virtual|**az140-adds-vnet11**|
   |Subred|**hp1-Subnet (10.0.1.0/24)**|
   |Grupo de seguridad de red|**Basic**|
   |Puertos de entrada públicos|**No**|
   |Seleccionar el directorio al que quieras unirte|**Active Directory**|
   |UPN de unión a un dominio de AD|**student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|
   |Confirmar contraseña|**Pa55w.rd1234**|
   |Especificar el nombre de dominio o la unidad|**Sí**|
   |Dominio al que deseas unirte|**adatum.com**|
   |Ruta de acceso de unidad organizativa|**OU=WVDInfra,DC=adatum,DC=com**|   
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|
   |Confirmar contraseña|**Pa55w.rd1234**|

   > **Nota**: Como es probable que haya observado, es posible cambiar la imagen y el prefijo de las máquinas virtuales a medida que agrega hosts de sesión al grupo existente. En general, esto no se recomienda a menos que planee reemplazar todas las máquinas virtuales del grupo. 

1. En la pestaña **Revisar y crear** de la hoja **Agregar máquinas virtuales a un grupo de hosts**, selecciona **Crear**.

   > **Nota**: espera a que la implementación se complete antes de avanzar a la siguiente tarea. Esto puede tardar unos 10 minutos. 

#### Tarea 4: Configuración de grupos de aplicaciones de Azure Virtual Desktop

1. Dentro de la sesión Bastion a **az140-dc-vm11**, en la ventana del navegador web que muestra el portal Azure, busca y selecciona **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, selecciona **Grupos de aplicaciones**.
1. En la hoja **Azure Virtual Desktop \| Grupos de aplicaciones**, busca el grupo de aplicaciones de escritorio existente y generado automáticamente **az140-21-hp1-DAG** y selecciónalo. 
1. En la hoja **az140-21-hp1-DAG**, selecciona **Asignaciones**.
1. En la hoja **az140-21-hp1-DAG \| Asignaciones**, selecciona **+ Agregar**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, selecciona **Grupos**, selecciona **az140-wvd-pooled** y haz clic en **Seleccionar**.
1. Vuelve a la hoja **Azure Virtual Desktop \| Grupos de aplicaciones** y selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la hoja **Crear un grupo de aplicaciones**, especifica la siguiente configuración y selecciona **Siguiente: Aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
   |Grupo de recursos|**az140-21-RG**|
   |Grupo de hosts|**az140-21-hp1**|
   |Tipo de grupo de aplicaciones|**Aplicación remota**|
   |Nombre del grupo de aplicaciones|**az140-21-hp1-Office365-RAG**|

1. En la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicación**, especifica la siguiente configuración y selecciona **Revisar y agregar **y, a continuación, selecciona **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Menú Inicio**|
   |Aplicación|**Word**|
   |Descripción|**Microsoft Word**|
   |Requiere línea de comandos|**No**|

1. De vuelta en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicación**, especifica la siguiente configuración y selecciona **Revisar y agregar **y, a continuación, selecciona **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Menú Inicio**|
   |Aplicación|**Excel**|
   |Descripción|**Microsoft Excel**|
   |Requiere línea de comandos|**No**|

1. De vuelta en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicación**, especifica la siguiente configuración y selecciona **Revisar y agregar **y, a continuación, selecciona **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Menú Inicio**|
   |Aplicación|**PowerPoint**|
   |Descripción|**Microsoft PowerPoint**|
   |Requiere línea de comandos|**No**|

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Asignaciones >**.
1. En la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, selecciona **Grupos**, luego selecciona **az140-wvd-remote-app** y haz clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Área de trabajo >**.
1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, especifica la siguiente configuración y selecciona **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear grupo de aplicaciones**, selecciona **Crear**.

   > **Nota**: espera a que se cree el grupo de aplicaciones. Debería tardar menos de un minuto. 

   > **Nota**: ahora crearás un grupo de aplicaciones basado en la ruta de acceso del archivo como origen de la aplicación.

1. Dentro de la sesión de Bastion a **az140-dc-vm11**, busca y selecciona **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, selecciona **Grupos de aplicaciones**.
1. En la hoja **Azure Virtual Desktop \| Grupos de aplicaciones**, selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la hoja **Crear un grupo de aplicaciones**, especifica la siguiente configuración y selecciona **Siguiente: Aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
   |Grupo de recursos|**az140-21-RG**|
   |Grupo de hosts|**az140-21-hp1**|
   |Tipo de grupo de aplicaciones|**RemoteApp**|
   |Nombre del grupo de aplicaciones|**az140-21-hp1-Utilities-RAG**|

1. En la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicación**, en la pestaña **Conceptos básicos**, especifica la siguiente configuración y selecciona **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Origen de aplicación|**Ruta de acceso del archivo**|
   |Ruta de acceso de la aplicación|**C:\Windows\system32\cmd.exe**|
   |Identificador de la aplicación|**Símbolo del sistema**|
   |Nombre para mostrar|**Símbolo del sistema**|
   |Descripción|**Símbolo del sistema de Windows**|
   |Requiere línea de comandos|**No**|

1. En la pestaña **Icono**, especifica la siguiente configuración y selecciona **Revisar + agregar**y, a continuación, selecciona **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Ruta de icono|**C:\Windows\system32\cmd.exe**|
   |Índice de icono|0|

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Asignaciones >**.
1. En la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, selecciona **grupos**, selecciona **az140-wvd-remote-app** y **az140-wvd-admins** y haz clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Área de trabajo >**.
1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, especifica la siguiente configuración y selecciona **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear grupo de aplicaciones**, selecciona **Crear**.

#### Tarea 5: Configuración de áreas de trabajo de Azure Virtual Desktop

1. Dentro de la sesión de Bastion a **az140-dc-vm11**, en la ventana de Microsoft Edge que muestra el portal Azure, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, selecciona **Áreas de trabajo**.
1. En la hoja **Azure Virtual Desktop \| Áreas de trabajo**, selecciona **+ Crear**. 
1. En la pestaña **Datos básicos** de la hoja **Crear un área de trabajo**, especifica la siguiente configuración y selecciona **Siguiente: Grupos de aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
   |Grupo de recursos|**az140-21-RG**|
   |Nombre del área de trabajo|**az140-21-ws1**|
   |Nombre descriptivo|**az140-21-ws1**|
   |Ubicación|Nombre de la región de Azure en la que implementaste los recursos en el primer ejercicio de este laboratorio o región cercana a la misma|

1. En la pestaña **Grupos de aplicaciones** de la hoja **Crear un área de trabajo**, especifica la siguiente configuración:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**Sí**|

1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, selecciona **+ Registrar grupos de aplicaciones**.
1. En la hoja **Agregar grupos de aplicaciones**, selecciona el signo más junto a las entradas **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** y **az140-21-hp1-Utilities-RAG** y haz clic en **Seleccionar**. 
1. De nuevo en la pestaña **Grupos de aplicaciones** de la hoja **Crear un área de trabajo**, selecciona **Revisar y crear**.
1. En la pestaña **Revisar y crear** de la hoja **Crear un área de trabajo**, selecciona **Crear**.

### Ejercicio 2: Validación de un entorno de Azure Virtual Desktop
  
Las tareas principales de este ejercicio son las siguientes:

1. Instalación del cliente Escritorio remoto de Microsoft (MSRDC) en un equipo con Windows 10
1. Suscripción a un área de trabajo de Azure Virtual Desktop
1. Prueba de aplicaciones de Azure Virtual Desktop

#### Tarea 1: Instalación del cliente Escritorio remoto de Microsoft (MSRDC) en un equipo con Windows 10

1. Dentro de la sesión Bastion a **az140-dc-vm11**, en la ventana del navegador que muestra el Azure portal, busque y seleccione **Máquinas virtuales** y, en el panel **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11**.
1. En la hoja **az140-cl-vm11**, desplácese hacia abajo hasta la sección **Operaciones** y seleccione **Ejecutar comando**. 
1. En la hoja **Ejecutar comandos de \|az140-cl-vm11**, seleccione **EnableRemotePS** y seleccione **Ejecutar**. 

   > **Nota**: Espere a que el comando se complete antes de avanzar al siguiente paso. Esto puede tardar 1 minuto. Puede obtener errores de texto rojo en relación al perfil público que se usa y no el perfil de dominio; si es así, puede omitirlos e ir al paso siguiente.

1. En la sesión de Bastion a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para agregar todos los miembros de **ADATUM\\az140-wvd-users** al grupo local **Usuarios de Escritorio remoto** en la máquina virtual Azure **az140-cl-vm11** que ejecuta Windows 10 que implementó en el laboratorio **Preparación de la implementación de Azure Virtual Desktop (AD DS)**.

   ```powershell
   $computerName = 'az140-cl-vm11'
   Invoke-Command -ComputerName $computerName -ScriptBlock {Add-LocalGroupMember -Group 'Remote Desktop Users' -Member 'ADATUM\az140-wvd-users'}
   ```

1. Cambie al equipo de laboratorio y desde él, en la ventana del explorador donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11**.
1. En el panel **az140-cl-vm11**, seleccione **Conectar**, en el menú desplegable, seleccione **Conectar a través de Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**Student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|

   > **Nota** El inicio de sesión inicial puede tardar entre 5 y 10 minutos en completarse.

1. Dentro de la sesión de Bastion a **az140-cl-vm11**, inicie Microsoft Edge y vaya a la [página de descarga del cliente Windows Desktop](https://go.microsoft.com/fwlink/?linkid=2068602) y, cuando se le solicite, seleccione **Ejecutar** para iniciar su instalación. En la página **Ámbito de instalación** del asistente de **Instalación de Escritorio remoto**, seleccione la opción **Instalar para todos los usuarios de esta máquina** y haga clic en **Instalar**. Si el Control de cuentas de usuario le solicita las credenciales administrativas, autentíquese mediante el nombre de usuario de **ADATUM\\Student** con **Pa55w.rd1234** como contraseña.
1. Una vez completada la instalación, asegúrese de que la casilla **Iniciar Escritorio remoto cuando se cierre la instalación** esté activada y haga clic en **Finalizar** para iniciar el cliente de Escritorio remoto.

#### Tarea 2: Suscripción a un área de trabajo de Azure Virtual Desktop

1. En la ventana del cliente de **Escritorio remoto**, seleccione **Suscribirse** y, cuando se le solicite, inicie sesión con las credenciales **aduser1**, proporcionando su atributo userPrincipalName que identificó anteriormente en este laboratorio y la contraseña que estableció al crear esta cuenta.

   > **Nota**: Como alternativa, en la ventana del cliente **Escritorio remoto**, seleccione **Suscribirse con dirección URL**, y, en el panel **Suscribirse a un área de trabajo**, en **URL del correo electrónico o área de trabajo**, escriba **https://client.wvd.microsoft.com/api/arm/feeddiscovery**, seleccione **Siguiente** y, una vez que se le solicite, inicie sesión con las credenciales de **aduser1** (con el atributo userPrincipalName como el nombre de usuario y la contraseña que estableció al crear esta cuenta). 

1. Asegúrate de que la página **Escritorio remoto** muestra la conexión SessionDesktop incluida en el grupo de aplicaciones de escritorio az140-21-hp1-DAG generado automáticamente, publicado en el área de trabajo y asociado a la cuenta de usuario **aduser1** mediante su pertenencia al grupo. 

   > **Nota**: Esto es algo esperado, ya que el **tipo de grupo de aplicaciones preferido** del grupo de hosts está establecido actualmente en **Escritorio**.

#### Tarea 3: Prueba de aplicaciones de Azure Virtual Desktop

1. Dentro de la sesión Bastion a **az140-cl-vm11**, en la ventana Cliente de **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **SessionDesktop** y compruebe que inicia una sesión de Escritorio remoto. 

   > **Nota**: Inicialmente, la aplicación puede tardar unos minutos en iniciarse pero, posteriormente, el inicio de la aplicación debe ser mucho más rápido.

   > **Nota**: Si aparece el mensaje de solicitud de inicio de sesión **Le damos la bienvenida a Microsoft Teams**, ciérrelo.

1. En la sesión **Escritorio remoto**, haga clic con el botón derecho en **Inicio**, seleccione **Ejecutar**, en el cuadro de texto **Abrir** del cuadro de diálogo **Ejecutar**, escriba **cmd** y seleccione **Aceptar**. 
1. Dentro de la sesión de **Escritorio remoto**, en el símbolo del sistema, escriba **hostname** y pulse la tecla **Intro** para mostrar el nombre del ordenador en el que se está ejecutando la sesión de Escritorio remoto.
1. Compruebe que el nombre mostrado sea **az140-21-p1-0**, **az140-21-p1-1** o **az140-21-p1-2**.
1. En el Símbolo del sistema, escriba **logoff** y presione la tecla **Intro** para cerrar sesión desde la sesión del Escritorio remoto.

   > **Nota**: A continuación, modificarás el **tipo de grupo de aplicaciones preferido** al establecerlo en **RemoteApp**.

1. Desde el equipo de laboratorio, cambie la sesión de Bastion a **az140-dc-vm11**.
1. Dentro de la sesión Bastion a **az140-dc-vm11**, en la ventana del navegador web que muestra el portal Azure, busca y selecciona **Azure Virtual Desktop** y, en el panel **Azure Virtual Desktop**, en la barra de menú vertical, en la **sección Administrar**, selecciona **Grupos de hosts**.
1. En la hoja **Grupos de hosts de \|Azure Virtual Desktop**, en la lista de grupos de hosts, selecciona **az140-21-hp1**.
1. En el panel **az140-21-hp1**, en la barra de menú vertical, en la sección **Configuración**, selecciona **Propiedades**; en el **tipo de grupo de aplicaciones preferido**, selecciona **Aplicación remota** y, luego, selecciona **Guardar**. 
1. Desde el equipo de laboratorio, cambia a la sesión de Bastion a **az140-cl-vm11**.
1. En la sesión de Bastion a **az140-cl-vm11**, en la ventana del cliente **Escritorio remoto**, selecciona el símbolo de puntos suspensivos en la esquina superior derecha y, en el menú desplegable, selecciona **Actualizar**.
1. Comprueba que la página **Escritorio remoto** muestra las aplicaciones individuales que se incluyen en los dos grupos de aplicaciones por ti creados y publicados en el área de trabajo, que también están asociados a la cuenta de usuario **aduser1** mediante su pertenencia a grupos. 

   > **Nota**: Esto es algo esperado, ya que el **tipo de grupo de aplicaciones preferido** del grupo de hosts está establecido ahora en **RemoteApp**.

1. En la sesión de Bastion a **az140-cl-vm11**, en la ventana de cliente de **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **Símbolo del sistema** y compruebe que inicia una ventana de **Símbolo del sistema**. Cuando se le pida que se autentique, escriba la contraseña que estableció al crear la cuenta de usuario **aduser1**, active la casilla **Recordarme** y seleccione **Aceptar**.
1. En el Símbolo del sistema, escriba **hostname** y presione la tecla **Intro** para mostrar el nombre del equipo en el que se ejecuta el Símbolo del sistema.

   > **Nota**: Compruebe que el nombre mostrado sea **az140-21-p1-0**, **az140-21-p1-1** o **az140-21-p1-2**, en lugar de **az140-cl-vm11**.

1. En el Símbolo del sistema, escriba **logoff** y presione la tecla **Intro** para cerrar sesión desde la sesión de aplicación remota actual.

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
