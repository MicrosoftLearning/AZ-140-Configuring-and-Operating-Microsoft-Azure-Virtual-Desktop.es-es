---
lab:
  title: "Laboratorio: Implementación y administración de almacenamiento para AVD (Microsoft\_Entra\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratorio: Implementación y administración de almacenamiento para AVD (Microsoft Entra DS)
# Manual de laboratorio para alumnos

## Dependencias de laboratorio

- Una suscripción de Azure
- Una cuenta de Microsoft o una cuenta de Microsoft Entra con el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure y con el rol Propietario o Colaborador en la suscripción de Azure
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (Microsoft Entra DS)**

## Tiempo estimado

30 minutos

## Escenario del laboratorio

Debe implementar y administrar el almacenamiento de una implementación de Azure Virtual Desktop en un entorno de Microsoft Entra DS.

## Objetivos
  
Después de completar este laboratorio, podrá:

- Configurar Azure Files para almacenar contenedores de perfiles de Azure Virtual Desktop en un entorno de Microsoft Entra DS

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Configurar Azure Files para almacenar contenedores de perfiles de Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Creación de una cuenta de Azure Storage
1. Crear un recurso compartido de Azure Files
1. Habilitar la autenticación de Microsoft Entra DS para la cuenta de Azure Storage 
1. Configurar los permisos del recurso compartido de Azure Files
1. Configurar los permisos de nivel de archivo o directorio de Azure Files

#### Tarea 1: Crear una cuenta de Azure Storage.

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio.
1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11a**. Se abrirá la hoja **az140-cl-vm11a**.
1. En la hoja **az140-cl-vm11a**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-cl-vm11a \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Valor|
   |---|---|
   |Nombre de usuario|**aadadmin1@adatum.com**|
   |Contraseña|La contraseña definida anteriormente|

1. En la sesión de Bastion a la máquina virtual de Azure **az140-cl-vm11a**, inicie Microsoft Edge, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión proporcionando el nombre principal de usuario de la cuenta de usuario **aadadmin1**, así como la contraseña que estableció al crear esta cuenta.

   >**Nota**: Puede identificar el atributo de nombre principal de usuario (UPN) de la cuenta **aadadmin1**; para ello, revise su cuadro de diálogo de propiedades desde la consola de Usuarios y equipos de Active Directory, o bien vuelva al equipo de laboratorio y revise sus propiedades desde la hoja del inquilino de Microsoft Entra en Azure Portal.

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de Microsoft Edge en la que se muestra Azure Portal, busque y seleccione **Cuentas de almacenamiento** y, en la hoja **Cuentas de almacenamiento**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear cuenta de almacenamiento**, configure las siguientes opciones (deje las demás con los valores predeterminados):

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
   |Resource group|nombre de un nuevo grupo de recursos **az140-22a-RG**.|
   |Nombre de la cuenta de almacenamiento|cualquier nombre único global de entre 3 y 15 caracteres, que conste de letras en minúsculas y dígitos, y que empiece por una letra.|
   |Location|nombre de una región de Azure que hospeda el entorno de laboratorio de Azure Virtual Desktop.|
   |Rendimiento|**Estándar**|
   |Replicación|**Almacenamiento con redundancia local (LRS)**|

   >**Nota**: Asegúrese de que el nombre de cuenta de almacenamiento no supere los 15 caracteres. El nombre se usará para crear una cuenta de equipo en el dominio de Active Directory Domain Services (AD DS) integrado con el inquilino de Microsoft Entra asociado a la suscripción de Azure que incluye la cuenta de almacenamiento. Esto permitirá la autenticación basada en AD DS al acceder a los recursos compartidos de archivos hospedados en esta cuenta de almacenamiento.

1. En la pestaña **Aspectos básicos** de la hoja **Crear cuenta de almacenamiento**, seleccione **Revisar y crear**, espere a que se complete el proceso de validación y, luego, seleccione **Crear**.

   >**Nota**: Espere a que se cree la cuenta de almacenamiento. Debe tardar aproximadamente dos minutos.

#### Tarea 2: Crear un recurso compartido de Azure Files

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de Microsoft Edge en la que se muestra Azure Portal, vuelva a la hoja **Cuentas de almacenamiento** y seleccione la entrada que representa la cuenta de almacenamiento recién creada.
1. En la hoja Cuenta de almacenamiento, en el menú vertical situado en el lado izquierdo, en la sección **Almacenamiento de datos**, seleccione **Recursos compartidos de archivos** y, después, **+ Recurso compartido de archivos**.
1. En la hoja **Nuevo recurso compartido**, especifique las opciones de configuración siguientes y seleccione **Crear** (deje las demás con los valores predeterminados):

   |Configuración|Valor|
   |---|---|
   |Nombre|**az140-22a-profiles**|

#### Tarea 3: Habilitar la autenticación de Microsoft Entra DS para la cuenta de Azure Storage

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana Microsoft Edge, en Azure Portal, en la hoja en la que se muestran las propiedades de la cuenta de almacenamiento que creó en la tarea anterior, en el menú vertical del lado izquierdo, en la sección **Almacenamiento de datos**, seleccione **Recursos compartidos de archivos**. 
1. En la sección **Configuración del recurso compartido de archivos**, junto a la etiqueta **Active Directory**, seleccione el vínculo **No configurado**.
1. En la sección **Enable an Active Directory source** (Habilitar un origen de Active Directory), en el rectángulo con la etiqueta **Azure Active Directory Domain Services**, seleccione **Configurar**.
1.  En la hoja **Acceso basado en identidad**, seleccione la opción **Habilitado** y luego **Guardar**.

#### Tarea 4: Configurar los permisos basados en RBAC de Azure Files

1. En la sesión de Bastion a **az140-cl-vm11a**, en la ventana de Microsoft Edge en la que se muestra Azure Portal, en la hoja en la que se muestran las propiedades de la cuenta de almacenamiento que creó anteriormente en este ejercicio, en el menú vertical del lado izquierdo, en la sección **Almacenamiento de datos**, seleccione **Recursos compartidos de archivos** y, en la lista de recursos compartidos, seleccione la entrada **az140-22a-profiles**.
1. En la hoja **az140-22a-profiles**, en el menú vertical del lado izquierdo, seleccione **Control de acceso (IAM)**.
1. En la hoja **az140-22a-profiles \| Control de acceso (IAM)**, seleccione **+ Agregar** y, en el menú desplegable, seleccione **Agregar asignación de roles.**
1. En la hoja **Agregar asignación de roles**, seleccione **Colaborador de recursos compartidos de SMB de datos de archivos de almacenamiento** y luego **Siguiente**:
1. En la hoja **Miembros**, seleccione  **Asignar acceso a** y luego haga clic en **+ Seleccionar miembros**.
1. En el panel **Select Members**, (Seleccionar miembros) en el cuadro de texto **Select** (Seleccionar), escriba **az140-wvd-ausers** y luego haga clic en **Select** (Seleccionar).
1. En la hoja **Miembros**, seleccione **Revisar y asignar** dos veces.
1. Repita los pasos del 3 al 8 anteriores y especifique la configuración siguiente:

   |Configuración|Valor|
   |---|---|
   |Role|**Colaborador elevado de recursos compartidos de SMB de datos de archivos de Storage**|
   |Seleccionar|**az140-wvd-aadmins**|

   > **Nota**: Usará la cuenta de usuario **aadadmin1**, que es miembro del grupo **az140-wvd-aadmins** para configurar los permisos del recurso compartido de archivos. 

#### Tarea 5: Configurar los permisos de nivel de archivo o directorio de Azure Files

1. En la sesión de Bastion a **az140-cl-vm11a**, inicie **Símbolo del sistema** y, desde la ventana **Símbolo del sistema**, ejecute lo siguiente para asignar una unidad al recurso compartido de destino (reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento):

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-22a-profiles
   ```

1. En la sesión de Bastion a **az140-cl-vm11a**, abra Explorador de archivos, vaya a la unidad Z: recién asignada, muestre su cuadro de diálogo **Propiedades**, seleccione la pestaña **Seguridad**, **Editar** y **Agregar**; en el cuadro de diálogo **Seleccionar usuarios, equipos, cuentas de servicio y grupos**, asegúrese de que el cuadro de texto **Desde esta ubicación** contiene la entrada **adatum.com**; en el cuadro de texto **Escriba el nombre del objeto que desea seleccionar**, escriba **az140-wvd-ausers** y haga clic en **Aceptar**.
1. De nuevo en la pestaña **Seguridad** del cuadro de diálogo en el que se muestran los permisos de la unidad asignada, asegúrese de que la entrada **az140-wvd-ausers** está seleccionada, seleccione la casilla **Modificar** de la columna **Permitir**, haga clic en **Aceptar**, revise el mensaje que se muestra en el cuadro de texto **Seguridad de Windows** y haga clic en **Sí**. 
1. De nuevo en la pestaña **Seguridad** del cuadro de diálogo en el que se muestran los permisos de la unidad asignada, seleccione **Editar** y **Agregar**; en el cuadro de diálogo **Select Users, Computers, Service Accounts, and Groups** (Seleccionar usuarios, equipos, cuentas de servicio y grupos), asegúrese de que el cuadro de texto **From this location** (Desde esta ubicación) contiene la entrada **adatum.com**; en el cuadro de texto **Escriba el nombre del objeto que desea seleccionar**, escriba **az140-wvd-aadmins** y haga clic en **Aceptar**.
1. De nuevo en la pestaña **Seguridad** del cuadro de diálogo en el que se muestran los permisos de la unidad asignada, asegúrese de que la entrada **az140-wvd-aadmins** está seleccionada, seleccione la casilla **Control total** de la columna **Permitir** y haga clic en **Aceptar**. 
1. En la pestaña **Seguridad** del cuadro de diálogo en el que se muestran los permisos de la unidad asignada, seleccione **Editar**, en la lista de grupos y nombres de usuario, seleccione la entrada **Usuarios autenticados** y luego **Quitar**.
1. Mientras sigue en la pantalla Editar, en la lista de grupos y nombres de usuario, seleccione la entrada **Usuarios** y luego **Quitar**, haga clic en **Aceptar** y, después, haga clic en **Aceptar** dos veces para completar el proceso. 

   >**Nota**: Como alternativa, podría establecer permisos mediante la utilidad de la línea de comandos **icacls**. 
