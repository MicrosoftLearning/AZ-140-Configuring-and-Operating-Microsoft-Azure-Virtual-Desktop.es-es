---
lab:
  title: "Laboratorio: Creación y configuración de grupos de hosts y hosts de sesión (Microsoft\_Entra\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratorio: Creación y configuración de grupos de hosts y hosts de sesión (Microsoft Entra DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure
- Una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure y con el rol Propietario o Colaborador en la suscripción de Azure
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (Microsoft Entra DS)**

## Tiempo estimado

60 minutos

## Escenario del laboratorio

Debe crear y configurar grupos de hosts y hosts de sesión en un entorno de Azure Active Directory Domain Services (Microsoft Entra DS).

## Objetivos
  
Después de completar este laboratorio, podrá:

- Configurar un entorno de Azure Virtual Desktop en un dominio de Microsoft Entra DS. 
- Validar un entorno de Azure Virtual Desktop en un dominio de Microsoft Entra DS. 

## Archivos de laboratorio

- Ninguno 

## Instrucciones

### Ejercicio 1: Configuración de un entorno de Azure Virtual Desktop
  
Las tareas principales de este ejercicio son las siguientes:

1. Preparación del dominio de AD DS y la suscripción de Azure para la implementación de un grupo de hosts de Azure Virtual Desktop
1. Implementación de un grupo de hosts de Azure Virtual Desktop
1. Configuración de grupos de aplicaciones de Azure Virtual Desktop
1. Configuración de áreas de trabajo de Azure Virtual Desktop

#### Tarea 1: Preparación del dominio de AD DS y la suscripción de Azure para la implementación de un grupo de hosts de Azure Virtual Desktop

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11a**. Se abrirá la hoja **az140-cl-vm11a**.
1. En la hoja **az140-cl-vm11a**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-cl-vm11a \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**aadadmin1@adatum.com**|
   |Contraseña|Contraseña definida en el laboratorio anterior|

1. En el Bastion a la máquina virtual de Azure **az140-cl-vm11a**, inicie Microsoft Edge, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión proporcionando el nombre principal de usuario de la cuenta de usuario **aadadmin1**, así como la contraseña que estableció al crear esta cuenta.

   >**Nota**: Puede identificar el atributo de nombre principal de usuario (UPN) de la cuenta **aadadmin1**; para ello, revise su cuadro de diálogo de propiedades desde la consola de Usuarios y equipos de Active Directory, o bien vuelva al equipo de laboratorio y revise sus propiedades desde la hoja del inquilino de Microsoft Entra en Azure Portal.

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de Microsoft Edge que muestra Azure Portal, abra una sesión de PowerShell en el **Cloud Shell** y ejecute el siguiente registro del proveedor de recursos **Microsoft.DesktopVirtualization**:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Redes virtuales** y, en la hoja **Redes virtuales**, seleccione la entrada **az140-aadds-vnet11a**. 
1. En la hoja **az140-aadds-vnet11a**, seleccione **Subredes**,y en la hoja **Subredes**, seleccione **+ Subred**; en la hoja **Agregar subred**, en el cuadro de texto **Nombre**, escriba **hp1-Subnet**, deje todas las demás opciones con sus valores predeterminados y seleccione **Guardar**. 

#### Tarea 2: Implementación de un grupo de hosts de Azure Virtual Desktop

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop**; en la hoja **Azure Virtual Desktop**, en el menú vertical del lado izquierdo, en la sección **Administrar**, seleccione **Grupos de hosts** y, en la hoja **Grupos de hosts de \|Azure Virtual Desktop**, seleccione **+ Crear**. 
1. En la pestaña **Datos básicos** de la hoja **Crear un grupo de hosts**, especifica la siguiente configuración y selecciona **Siguiente: Máquinas virtuales >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|nombre de un nuevo grupo de recursos **az140-21a-RG**.|
   |Nombre del grupo de hosts|**az140-21a-hp1**|
   |Ubicación|Nombre de la región de Azure en la que implementaste la instancia de Microsoft Entra DS anteriormente en este laboratorio|
   |Entorno de validación|**No**|
   |Tipo de grupo de aplicaciones preferido|**Dispositivo de escritorio**|
   |Tipo de grupo de hosts|**Agrupado**|
   |Límite máximo de sesión|**12**
          |
   |Algoritmo de equilibrio de carga|**Equilibrio de carga en amplitud**|

   > **Nota**: si un usuario tiene aplicaciones de Escritorio y RemoteApp publicadas, el tipo de grupo de aplicaciones preferido determina cuál de ellas aparecerá en su fuente.

1. En la pestaña **Máquinas virtuales** de la hoja **Crear un grupo de hosts**, especifica la configuración siguiente (deja los otros valores con su configuración predeterminada) y selecciona **Siguiente: Área de trabajo >** (sustituye el marcador de posición *<Azure_AD_domain_name>* con el nombre del inquilino de Microsoft Entra asociado con la suscripción con la que implementaste la instancia de Microsoft Entra DS y sustituye el marcador de posición `<password>` con la contraseña que estableciste al crear la cuenta aadadmin1):

   > **Nota**: asegúrate de recordar la contraseña que has usado, ya que la necesitaremos más adelante en este y en los laboratorios posteriores.

   |Configuración|Valor|
   |---|---|
   |Agregar máquinas virtuales|**Sí**|
   |Grupo de recursos|**El valor predeterminado es el mismo que el del grupo de hosts**|
   |Prefijo de nombre|**az140-21-p1**|
   |Ubicación de la máquina virtual|nombre de la región de Azure en la que implementaste los recursos en el primer ejercicio de este laboratorio|
   |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
   |Tipo de seguridad|**Máquina virtual de inicio seguro**|
   |Imagen|**Windows 11 Enterprise para sesiones múltiples y Aplicaciones de Microsoft 365, versión 22H2**|
   |Tamaño de la máquina virtual|**Standard DC2s_v3**|
   |Número de máquinas virtuales|**2**|
   |Tipo de disco del sistema operativo|**SSD estándar**|
   |Red virtual|**az140-aadds-vnet11a**|
   |Subnet|**hp1-Subnet (10.10.1.0/24)**|
   |Grupo de seguridad de red|**Basic**|
   |Puertos de entrada públicos|**No**|
   |Seleccionar el directorio al que quieras unirte|**Active Directory**|
   |UPN de unión a un dominio de AD|**aadadmin1@adatum.com**|
   |Contraseña|Utilice la contraseña de aadadmin1|
   |Especifique el nombre de dominio o la unidad|**Sí**|
   |Dominio al que desea unirse|**adatum.com**|
   |Ruta de acceso de unidad organizativa|**OU=equipos AADDC,DC=adatum,DC=com**|
   |Nombre de usuario de la cuenta de administrador de máquina virtual|**estudiante**|
   |Contraseña de la cuenta de administrador de máquina virtual|**Pa55w.rd1234**|

1. En la pestaña **Área de trabajo** de la hoja **Crear un grupo de hosts**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicación de escritorio|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear un grupo de hosts**, seleccione **Crear**.

   > **Nota**: Espere a que la implementación se complete. Este proceso tardará aproximadamente 15 minutos.

#### Tarea 3: Configuración de grupos de aplicaciones de Azure Virtual Desktop

1. En la sesión de Bastion a **az140-cl-vm11a**, en Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, seleccione **Grupos de aplicaciones**.
1. En la hoja **Grupos de aplicaciones de \|Azure Virtual Desktop**, seleccione el grupo de aplicaciones de escritorio **az140-21a-hp1-DAG** generado automáticamente.
1. En la hoja **az140-21a-hp1-DAG**, en el menú vertical del lado izquierdo, en la sección **Administrar**, haga clic en **Asignaciones**.
1. En la hoja **Asignaciones de \|az140-21a-hp1-DAG**, seleccione **+ Agregar**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, seleccione **az140-wvd-apooled** y haga clic en **Seleccionar**.
1. Vuelva a la hoja **Grupos de aplicaciones de \|Azure Virtual Desktop** y seleccione **+ Crear**.
1. En la pestaña **Datos básicos** de la hoja **Crear un grupo de aplicaciones**, especifica la siguiente configuración y selecciona **Siguiente: Aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|**az140-21a-RG**|
   |Grupo de hosts|**az140-21a-hp1**|
   |Tipo de grupo de aplicaciones|**RemoteApp**|
   |Nombre del grupo de aplicaciones|**az140-21a-hp1-Office365-RAG**|

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

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Asignaciones >**.
1. En la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, seleccione **az140-wvd-aremote-app** y haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Área de trabajo >**.
1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear grupo de aplicaciones**, seleccione **Crear**.

   > **Nota**: Ahora creará un grupo de aplicaciones basado en la ruta de acceso del archivo como origen de la aplicación.

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana del explorador web que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, seleccione **Grupos de aplicaciones**.
1. En la hoja **Grupos de aplicaciones \|de Azure Virtual Desktop**, seleccione **+ Crear**. 
1. En la pestaña **Datos básicos** de la hoja **Crear un grupo de aplicaciones**, especifica la siguiente configuración y selecciona **Siguiente: Aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|**az140-21a-RG**|
   |Grupo de hosts|**az140-21a-hp1**|
   |Tipo de grupo de aplicaciones|**RemoteApp**|
   |Nombre del grupo de aplicaciones|**az140-21a-hp1-Utilities-RAG**|

1. En la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar aplicaciones**.
1. En la hoja **Agregar aplicaciones**, especifique la configuración siguiente y seleccione **Guardar**:

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

1. De nuevo en la pestaña **Aplicaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Asignaciones >**.
1. En la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, seleccione **+ Agregar usuarios o grupos de usuarios de Microsoft Entra**.
1. En la hoja **Seleccionar usuarios o grupos de usuarios de Microsoft Entra**, seleccione **az140-wvd-aremote-app** y **az140-wvd-aadmins** y haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Asignaciones** de la hoja **Crear un grupo de aplicaciones**, selecciona **Siguiente: Área de trabajo >**.
1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**No**|

1. En la pestaña **Revisar y crear** de la hoja **Crear grupo de aplicaciones**, seleccione **Crear**.

#### Tarea 4: Configuración de áreas de trabajo de Azure Virtual Desktop

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de Microsoft Edge que muestra Azure Portal, busque y seleccione **Azure Virtual Desktop** y, en la hoja **Azure Virtual Desktop**, seleccione **Áreas de trabajo**.
1. En la hoja **Áreas de trabajo de \|Azure Virtual Desktop**, seleccione **+ Crear**. 
1. En la pestaña **Datos básicos** de la hoja **Crear un área de trabajo**, especifica la siguiente configuración y selecciona **Siguiente: Grupos de aplicaciones >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|**az140-21a-RG**|
   |Nombre del área de trabajo|**az140-21a-ws1**|
   |Nombre descriptivo|**az140-21a-ws1**|
   |Location|nombre de la región de Azure en la que implementó recursos de este laboratorio||

1. En la pestaña **Grupos de aplicaciones** de la hoja **Crear un área de trabajo**, especifique la siguiente configuración:

   |Configuración|Valor|
   |---|---|
   |Registro de un grupo de aplicaciones|**Sí**|

1. En la pestaña **Área de trabajo** de la hoja **Crear un área de trabajo**, seleccione **+ Registrar grupos de aplicaciones**.
1. En la hoja **Agregar grupos de aplicaciones**, seleccione el signo más junto a **az140-21a-hp1-DAG**, **az140-21a-hp1-Office365-RAG** y **az140-21a-hp1-Utilities-RAG** y haga clic en **Seleccionar**. 
1. De nuevo en la pestaña **Grupos de aplicaciones** de la hoja **Crear un área de trabajo**, seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear** de la hoja **Crear un área de trabajo**, seleccione **Crear**.

### Ejercicio 2: Validación de un entorno de Azure Virtual Desktop
  
Las tareas principales de este ejercicio son las siguientes:

1. Instalación del cliente Escritorio remoto de Microsoft (MSRDC) en un equipo con Windows 10
1. Suscripción a un área de trabajo de Azure Virtual Desktop
1. Prueba de aplicaciones de Azure Virtual Desktop

#### Tarea 1: Instalación del cliente Escritorio remoto de Microsoft (MSRDC) en un equipo con Windows 10

1. En la sesión de Bastion a **az140-cl-vm11a**, inicie Microsoft Edge y vaya a la [página de descarga del cliente de escritorio de Windows](https://go.microsoft.com/fwlink/?linkid=2068602) y, cuando se le solicite, ejecute su instalación siguiendo las indicaciones siguientes. Seleccione la opción **Instalar para todos los usuarios de esta máquina**. 
1. Una vez completada la instalación, inicie el cliente de Escritorio remoto.

#### Tarea 2: Suscripción a un área de trabajo de Azure Virtual Desktop

1. En la ventana de cliente de **Escritorio remoto**, seleccione **Suscribirse** y, cuando se le solicite, inicie sesión con las credenciales de **aaduser1** (con el atributo userPrincipalName como nombre de usuario y la contraseña que estableció al crear esta cuenta). 

   > **Nota**: Como alternativa, en la ventana del cliente**Escritorio remoto**, seleccione **Suscribirse con dirección URL**, y, en el panel **Suscribirse a un área de trabajo**, en **URL del correo electrónico o área de trabajo**, escriba **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, seleccione **Siguiente** y, una vez que se le solicite, inicie sesión con las credenciales de **aaduser1** (con el atributo userPrincipalName como el nombre de usuario y **Pa55w.rd1234** como contraseña). 

   > **Nota**: El nombre principal de usuario de **aaduser1** debe tener el formato **aaduser1@***<Azure_AD_domain_name>*, donde el marcador de posición *<Azure_AD_domain_name>* coincide con el nombre del inquilino de Microsoft Entra asociado a la suscripción en la que implementó la instancia de Microsoft Entra DS.

1. En la ventana **Mantener sesión iniciada en todas las aplicaciones**, desactive la casilla **Permitir que mi organización administre el dispositivo** y seleccione **No, iniciar sesión solo en esta aplicación**. 
1. Asegúrate de que la página **Escritorio remoto** muestra la conexión SessionDesktop incluida en el grupo de aplicaciones de escritorio az140-21-hp1-DAG generado automáticamente, publicado en el área de trabajo y asociado a la cuenta de usuario **aduser1** mediante su pertenencia al grupo. 

   > **Nota**: Esto es algo esperado, ya que el **tipo de grupo de aplicaciones preferido** del grupo de hosts está establecido actualmente en **Escritorio**.

#### Tarea 3: Prueba de aplicaciones de Azure Virtual Desktop

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana del cliente **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **SessionDesktop** y compruebe que inicia sesión en Escritorio remoto. 

   > **Nota**: Inicialmente, la aplicación puede tardar unos minutos en iniciarse pero, posteriormente, el inicio de la aplicación debe ser mucho más rápido.

   > **Nota**: Si aparece el mensaje de solicitud de inicio de sesión **Le damos la bienvenida a Microsoft Teams**, ciérrelo. 

1. En la sesión **Escritorio remoto**, haga clic con el botón derecho en **Inicio**, seleccione **Ejecutar**, en el cuadro de texto **Abrir** del cuadro de diálogo **Ejecutar**, escriba **cmd** y seleccione **Aceptar**. 
1. Dentro de la sesión de **Escritorio remoto**, en el símbolo del sistema, escriba **hostname** y pulse la tecla **Intro** para mostrar el nombre del ordenador en el que se está ejecutando la sesión de Escritorio remoto.
1. Compruebe que el nombre mostrado sea **az140-21-p1-0**, **az140-21-p1-1** o **az140-21-p1-2**.
1. En el Símbolo del sistema, escriba **logoff** y presione la tecla **Intro** para cerrar sesión desde la sesión del Escritorio remoto.

   > **Nota**: A continuación, modificarás el **tipo de grupo de aplicaciones preferido** al establecerlo en **RemoteApp**.

1. Dentro de la sesión Bastion a **az140-cl-vm11a**, en la ventana del navegador web que muestra Azure Portal, busca y selecciona **Azure Virtual Desktop** y, en el panel **Azure Virtual Desktop**, en la barra de menú vertical, en la **sección Administrar**, selecciona **Grupos de hosts**.
1. En la hoja **Grupos de hosts de \|Azure Virtual Desktop**, en la lista de grupos de hosts, seleccione **az140-21-hp1**.
1. En el panel **az140-21-hp1**, en la barra de menú vertical, en la sección **Configuración**, selecciona **Propiedades**; en el **tipo de grupo de aplicaciones preferido**, selecciona **Aplicación remota** y, luego, selecciona **Guardar**. 
1. En la sesión de Bastion a **az140-cl-vm11**, en la ventana del cliente **Escritorio remoto**, selecciona el símbolo de puntos suspensivos en la esquina superior derecha y, en el menú desplegable, selecciona **Actualizar**.
1. Comprueba que la página **Escritorio remoto** muestra las aplicaciones individuales que se incluyen en los dos grupos de aplicaciones por ti creados y publicados en el área de trabajo, que también están asociados a la cuenta de usuario **aduser1** mediante su pertenencia a grupos. 

   > **Nota**: Esto es algo esperado, ya que el **tipo de grupo de aplicaciones preferido** del grupo de hosts está establecido ahora en **RemoteApp**.

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de cliente de **Escritorio remoto**, en la lista de aplicaciones, haga doble clic en **Símbolo del sistema** y compruebe que inicia una ventana de **Símbolo del sistema**. Cuando se le pida que se autentique, escriba la contraseña que estableció al crear la cuenta de usuario **aduser1**, active la casilla **Recordarme** y seleccione **Aceptar**.
1. En el Símbolo del sistema, escribe **logoff** y presiona la tecla **Intro** para cerrar sesión desde la sesión de aplicación remota actual.

### Ejercicio 3: Detención y desasignación de las máquinas virtuales de Azure aprovisionadas en el laboratorio

Las tareas principales de este ejercicio son las siguientes:

1. Detener y desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

>**Nota**: En este ejercicio, desasignaremos las máquinas virtuales de Azure aprovisionadas en este laboratorio para reducir al mínimo los cargos de proceso correspondientes.

#### Tarea 1: Desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

1. Cambie al equipo de laboratorio y, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell, en el panel de Cloud Shell, ejecute lo siguiente para mostrar todas las máquinas virtuales de Azure que hemos creado y usado en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG'
   ```

1. Desde la sesión de PowerShell, en el panel de Cloud Shell, ejecute lo siguiente para detener y desasignar todas las máquinas virtuales de Azure que hemos creado y usado en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro -NoWait). Aunque podrá ejecutar otro comando de PowerShell inmediatamente después en la misma sesión de PowerShell, las máquinas virtuales de Azure tardarán unos minutos en detenerse y desasignarse.

