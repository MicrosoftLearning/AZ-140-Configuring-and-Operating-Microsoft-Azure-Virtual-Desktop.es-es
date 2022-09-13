---
lab:
  title: "Laboratorio: Configuración de directivas de acceso condicional para WVD (AD\_DS)"
  module: 'Module 3: Manage Access and Security'
---

# <a name="lab---configure-conditional-access-policies-for-wvd-ad-ds"></a>Laboratorio: Configuración de directivas de acceso condicional para WVD (AD DS)
# <a name="student-lab-manual"></a>Manual de laboratorio para alumnos

## <a name="lab-dependencies"></a>Dependencias de laboratorio

- Una suscripción de Azure
- Una cuenta de Microsoft o una cuenta de Azure AD con el rol Administrador global en el inquilino de Azure AD asociado a la suscripción de Azure y con el rol Propietario o Colaborador en la suscripción de Azure.
- Haber completado el laboratorio **Preparación de una implementación de Azure Virtual Desktop (AD DS)**
- Haber completado el laboratorio **Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (AD DS)**

## <a name="estimated-time"></a>Tiempo estimado

60 minutos

## <a name="lab-scenario"></a>Escenario del laboratorio

Necesitamos controlar el acceso a una implementación de Azure Virtual Desktop en un entorno de Active Directory Domain Services (AD DS) por medio del acceso condicional de Azure Active Directory (Azure AD).

## <a name="objectives"></a>Objetivos
  
Después de completar este laboratorio, podrá:

- Preparar el acceso condicional basado en Azure Active Directory (Azure AD) para Azure Virtual Desktop
- Implementar el acceso condicional basado en Azure AD para Azure Virtual Desktop

## <a name="lab-files"></a>Archivos de laboratorio

- None 

## <a name="instructions"></a>Instructions

### <a name="exercise-1-prepare-for-azure-ad-based-conditional-access-for-azure-virtual-desktop"></a>Ejercicio 1: Preparar el acceso condicional basado en Azure AD para Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Configurar la licencia de Azure AD Premium P2
1. Configurar Azure AD Multi-Factor Authentication (MFA)
1. Registrar un usuario para Azure AD MFA
1. Configuración de la unión a Azure AD híbrido
1. Desencadenar una sincronización diferencial de Azure AD Connect

#### <a name="task-1-configure-azure-ad-premium-p2-licensing"></a>Tarea 1: Configurar la licencia de Azure AD Premium P2

>**Nota**: Para implementar el acceso condicional de Azure AD, se necesitan licencias Premium P1 o P2 de Azure AD. En este laboratorio usaremos una licencia de prueba de 30 días.

1. En el equipo de laboratorio, inicie un explorador web, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que va a usar en este laboratorio, así como el rol Administrador global en el inquilino de Azure AD asociado a la suscripción.
1. En Azure Portal, busque y seleccione **Azure Active Directory** para ir al inquilino de Azure AD asociado a la suscripción de Azure que estamos usando en este laboratorio.
1. En la barra de menús vertical izquierda de la hoja de Azure Active Directory, en la sección **Administrar**, seleccione **Usuarios**. 
1. En la hoja **Usuarios | Todos los usuarios (versión preliminar)** , seleccione **aduser5**.
1. En la barra de herramientas de la hoja **aduser5 | Perfil**, haga clic en **Editar**; en la sección **Configuración**, en la lista desplegable **Ubicación de uso**, seleccione el país donde se encuentra el entorno de laboratorio y, en la barra de herramientas, haga clic en **Guardar**.
1. En la hoja **aduser5 | Perfil**, en la sección **Identidad**, localice el nombre principal de usuario de la cuenta **aduser5**.

    >**Nota**: Registre este valor. Lo necesitará más adelante en este laboratorio.

1. En la hoja **Usuarios | Todos los usuarios (versión preliminar)** , seleccione la cuenta de usuario que usamos para iniciar sesión al principio de esta tarea y repita el paso anterior si la cuenta no tiene una **Ubicación de uso** asignada. 

    >**Nota**: La propiedad **Ubicación de uso** debe establecerse para poder asignar una licencia de Azure AD Premium P2 a cuentas de usuario.

1. En la hoja **Usuarios | Todos los usuarios (versión preliminar)** , seleccione la cuenta de usuario **aadsyncuser** y localice su nombre principal de usuario.

    >**Nota**: Registre este valor. Lo necesitará más adelante en este laboratorio.

1. En Azure Portal, regrese a la hoja **Información general** del inquilino de Azure AD y, en la barra de menús vertical de la izquierda, en la sección **Administrar**, haga clic en **Licencias**.
1. En la barra de menús vertical izquierda de la hoja **Licencias \| Información general**, en la sección **Administrar**, haga clic en **Todos los productos**.
1. En la barra de herramientas de la hoja **Licencias \| Todos los productos**, haga clic en **+ Probar/Comprar**.
1. En la hoja **Activar**, haga clic en **Prueba gratuita** en la sección **ENTERPRISE MOBILITY + SECURITY E5** y, a continuación, haga clic en **Activar**. 
1. Mientras está en la hoja **Licencias \| Información general**, actualice la ventana del explorador para confirmar que la activación se ha realizado correctamente. 
1. En la hoja **Licencias - Todos los productos**, seleccione la entrada **Enterprise Mobility + Security E5**. 
1. En la barra de herramientas la hoja **Enterprise Mobility + Security E5**, haga clic en **+ Asignar**.
1. En la hoja **Asignar licencia**, haga clic en **Agregar usuarios y grupos**; en la hoja **Agregar usuarios y grupos**, seleccione **aduser5** y su cuenta de usuario y haga clic en **Seleccionar**.
1. De nuevo en la hoja **Asignar licencia**, haga clic en **Opciones de asignación**; en la hoja **Opciones de asignación**, compruebe que todas las opciones están habilitadas, haga clic en **Revisar y asignar** y haga clic en **Asignar**.

#### <a name="task-2-configure-azure-ad-multi-factor-authentication-mfa"></a>Tarea 2: Configurar Azure AD Multi-Factor Authentication (MFA)

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, regrese a la hoja **Información general** del inquilino de Azure AD y, en el menú vertical de la izquierda, en la sección **Administrar**, haga clic en **Seguridad**.
1. En la hoja **Seguridad | Introducción**, en la sección **Proteger** del menú vertical de la izquierda, haga clic en **Identity Protection**.
1. En la hoja **Identity Protection | Información general**, en la sección **Proteger** del menú vertical de la izquierda, haga clic en **Directiva de registro de MFA** (si es necesario, actualice la página del explorador web).
1. En la hoja **Identity Protection | Directiva de registro de MFA**, en la sección **Asignaciones** de **Directiva de registro de autenticación multifactor**, haga clic en **Todos los usuarios**. En la pestaña **Incluir**, haga clic en la opción **Seleccionar individuos y grupos**; en **Seleccionar usuarios**, haga clic en **aduser5** y en **Seleccionar** y, a continuación, en la parte inferior de la hoja, establezca el conmutador **Aplicar directiva** en **Activado** y haga clic en **Guardar**.

#### <a name="task-3-register-a-user-for-azure-ad-mfa"></a>Tarea 3: Registrar un usuario para Azure AD MFA

1. En el equipo de laboratorio, abra una sesión de explorador web **InPrivate**, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión indicando el nombre principal de usuario de **aduser5** que localizamos anteriormente en este ejercicio y la contraseña que establecimos al crear esta cuenta de usuario.
1. Cuando aparezca el mensaje **Se necesita más información**, haga clic en **Siguiente**. Esto redirigirá automáticamente el explorador a la página de **Microsoft Authenticator**.
1. En la página **Comprobación de seguridad adicional**, en la sección **Paso 1: ¿Cómo desea que nos pongamos en contacto con usted?** , seleccione el método de autenticación de su preferencia y siga las instrucciones para completar el proceso de registro. 
1. En la página de Azure Portal, en la esquina superior derecha, haga clic en el icono que representa el avatar del usuario, haga clic en **Cerrar sesión** y cierre la ventana del explorador **InPrivate**. 

#### <a name="task-4-configure-hybrid-azure-ad-join"></a>Tarea 4: Configuración de la unión a Azure AD híbrido

> **Nota**: Esta funcionalidad se puede usar para implementar más seguridad al configurar el acceso condicional de los dispositivos en función de su estado de unión a Azure AD.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione **az140-dc-vm11**.
1. En la hoja **az140-dc-vm11**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-dc-vm11 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Value|
   |---|---|
   |Nombre de usuario|**Estudiante**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en el menú **Inicio**, expanda la carpeta **Azure AD Connect** y seleccione **Azure AD Connect**.
> **Nota:** Si aparece una ventana de error que indica que el servicio de sincronización no se está ejecutando, vaya a la ventana de comandos de PowerShell, escriba **Start-Service "ADSync"** e intente volver a realizar el paso 4.
3. En la página **Bienvenido a Azure AD Connect** de la ventana **Microsoft Azure Active Directory Connect**, seleccione **Configurar**.
4. En la página **Tareas adicionales** de la ventana **Microsoft Azure Active Directory Connect**, seleccione **Configurar opciones de dispositivo** y luego, **Siguiente**.
5. En la página **Información general** de la ventana **Microsoft Azure Active Directory Connect**, revise la información relativa a **Unión a Azure AD híbrido** y a **Escritura diferida de dispositivo** y seleccione **Siguiente**.
6. En la página **Conectar a Azure AD** de la ventana **Microsoft Azure Active Directory Connect**, autentíquese con las credenciales de la cuenta de usuario **aadsyncuser** que creamos en el ejercicio anterior y seleccione **Siguiente**.  

   > **Nota**: Proporcione el atributo userPrincipalName de la cuenta **aadsyncuser** que registramos anteriormente en este laboratorio y especifique la contraseña que establecimos al crear esta cuenta de usuario. 

1. En la página **Opciones de dispositivos** de la ventana **Microsoft Azure Active Directory Connect**, asegúrese de que la opción **Configurar la combinación de Azure AD híbrido** está seleccionada y seleccione **Siguiente**. 
1. En la página **Sistemas operativos de los dispositivos** de la ventana **Microsoft Azure Active Directory Connect**, active la casilla **Dispositivos unidos a dominios con Windows 10 o versiones posteriores** y seleccione **Siguiente**. 
1. En la página **Configuración del SCP** de la ventana **Microsoft Azure Active Directory Connect**, active la casilla situada junto a la entrada **adatum.com** y, en la lista desplegable **Servicio de autenticación**, seleccione la entrada **Azure Active Directory** y luego, **Agregar**. 
1. Cuando se le solicite, en el cuadro de diálogo **Credenciales de administrador de empresa**, especifique las siguientes credenciales y seleccione **Aceptar**:

   |Configuración|Value|
   |---|---|
   |Nombre de usuario|**ADATUM\Student**|
   |Contraseña|**Pa55w.rd1234**|

1. De nuevo en la página **Configuración del SCP** de la ventana **Microsoft Azure Active Directory Connect**, seleccione **Siguiente**.
1. En la página **Listo para configurar** de la ventana **Microsoft Azure Active Directory Connect**, seleccione **Configurar** y, una vez completada la configuración, seleccione **Salir**.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, inicie **Windows PowerShell ISE** como administrador.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para mover la cuenta de equipo **az140-cl-vm11** a la unidad organizativa **WVDClients**:

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en el menú **Inicio**, expanda la carpeta **Azure AD Connect** y seleccione **Azure AD Connect**.
1. En la página **Bienvenido a Azure AD Connect** de la ventana **Microsoft Azure Active Directory Connect**, seleccione **Configurar**.
1. En la página **Tareas adicionales** de la ventana **Microsoft Azure Active Directory Connect**, seleccione **Personalizar las opciones de sincronización** y luego, **Siguiente**.
1. En la página **Conectar a Azure AD** de la ventana **Microsoft Azure Active Directory Connect**, autentíquese con las credenciales de la cuenta de usuario **aadsyncuser** que creamos en el ejercicio anterior y seleccione **Siguiente**. 

   > **Nota**: Proporcione el atributo userPrincipalName de la cuenta **aadsyncuser** que registramos anteriormente en este laboratorio y especifique la contraseña que establecimos al crear esta cuenta de usuario. 

1. En la página **Conectar sus directorios** de la ventana **Microsoft Azure Active Directory Connect**, seleccione **Siguiente**.
1. En la página **Filtrado de dominios y unidades organizativas** de la ventana **Microsoft Azure Active Directory Connect**, asegúrese de que la opción **Sincronizar los dominios y las unidades organizativas seleccionados** está seleccionada, expanda el nodo **adatum.com**, asegúrese de que la casilla situada junto a la unidad organizativa **ToSync** está activada, active la casilla situada junto a la unidad organizativa **WVDClients** y seleccione **Siguiente**.
1. En la página **Características opcionales** de la ventana **Microsoft Azure Active Directory Connect**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la página **Listo para configurar** de la ventana **Microsoft Azure Active Directory Connect**, asegúrese de que la casilla **Inicie el proceso de sincronización cuando se complete la configuración** está seleccionada y seleccione **Configurar**.
1. Revise la información de la página **Configuración completada** y seleccione **Salir** para cerrar la ventana **Microsoft Azure Active Directory Connect**.

#### <a name="task-5-trigger-azure-ad-connect-delta-synchronization"></a>Tarea 5: Desencadenar una sincronización diferencial de Azure AD Connect

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, cambie a la ventana **Administrador: Windows PowerShell ISE**.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, desde el panel de la consola **Administrador: Windows PowerShell ISE**, ejecute lo siguiente para desencadenar la sincronización diferencial de Azure AD Connect:

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. En la sesión de Escritorio remoto a **az140-dc-vm11**, inicie Microsoft Edge y navegue hasta [Azure Portal](https://portal.azure.com). Cuando se le solicite, inicie sesión con las credenciales de Azure AD de la cuenta de usuario con el rol Administrador global en el inquilino de Azure AD asociado a la suscripción de Azure que estamos usando en este laboratorio.
1. En la sesión de Escritorio remoto a **az140-dc-vm11**, en la ventana de Microsoft Edge donde se muestra Azure Portal, busque y seleccione **Azure Active Directory** para ir al inquilino de Azure AD asociado a la suscripción de Azure que estamos usando en este laboratorio.
1. En la barra de menús vertical izquierda de la hoja de Azure Active Directory, en la sección **Administrar**, seleccione **Dispositivos**. 
1. En la hoja **Dispositivos | Todos los dispositivos**, revise la lista de dispositivos y compruebe que el dispositivo **az140-cl-vm11** aparece con la entrada **Unidos a Azure AD híbrido** en la columna **Tipo de unión**.

   > **Nota**: Es posible que haya que esperar unos minutos a que la sincronización surta efecto y el dispositivo aparezca en Azure Portal.

### <a name="exercise-2-implement-azure-ad-based-conditional-access-for-azure-virtual-desktop"></a>Ejercicio 2: Implementar el acceso condicional basado en Azure AD para Azure Virtual Desktop

Las tareas principales de este ejercicio son las siguientes:

1. Crear una directiva de acceso condicional basado en Azure AD para todas las conexiones de Azure Virtual Desktop
1. Probar la directiva de acceso condicional basado en Azure AD de todas las conexiones de Azure Virtual Desktop
1. Modificar la directiva de acceso condicional basado en Azure AD para excluir los equipos unidos a Azure AD híbrido del requisito de MFA
1. Probar la directiva de acceso condicional basado en Azure AD modificado

#### <a name="task-1-create-an-azure-ad-based-conditional-access-policy-for-all-azure-virtual-desktop-connections"></a>Tarea 1: Crear una directiva de acceso condicional basado en Azure AD para todas las conexiones de Azure Virtual Desktop

>**Nota**: En esta tarea, configuraremos una directiva de acceso condicional basado en Azure AD que requiere MFA para iniciar sesión en una sesión de Azure Virtual Desktop. La directiva también obligará a reautenticarse transcurridas las primeras 4 horas después de una autenticación correcta.

1. En el equipo de laboratorio, en el explorador web donde se muestra Azure Portal, regrese a la hoja **Información general** del inquilino de Azure AD y, en el menú vertical de la izquierda, en la sección **Administrar**, haga clic en **Seguridad**.
1. En la hoja **Seguridad \| Introducción**, en la sección **Proteger** del menú vertical de la izquierda, haga clic en **Acceso condicional**.
1. En la barra de herramientas de la hoja **Acceso condicional \| Directivas**, haga clic en **+ Nueva directiva** y, en el menú contextual, seleccione **Crear nueva directiva**.
1. En la hoja **Nuevo**, configure las siguientes opciones:

   - En el cuadro de texto **Nombre**, escriba **az140-31-wvdpolicy1**.
   - En la sección **Asignaciones**, seleccione la opción **Users or workload identities** (Usuarios o identidades de carga de trabajo); en la lista desplegable **What does this policy apply to?** (¿A qué se aplica esta directiva?), asegúrese de que **Usuarios y grupos** está seleccionado; en la sección **Seleccionar usuarios y grupos**, active la casilla **Usuarios y grupos** y, por último, en la hoja **Seleccionar**, haga clic en **aduser5** y luego, en **Seleccionar**.
   - En la sección **Asignaciones**, haga clic en **Aplicaciones en la nube o acciones**, asegúrese de que, en el conmutador **Seleccione a qué se aplica esta directiva**, la opción **Aplicaciones en la nube** está seleccionada; haga clic en la opción **Seleccionar aplicaciones**; en la hoja **Seleccionar**, active la casilla situada junto a la entrada **Azure Virtual Desktop** y haga clic en **Seleccionar**. 
   - En la sección **Asignaciones**, haga clic en **Condiciones** y en **Aplicaciones cliente**; en la hoja **Aplicaciones cliente**, establezca el conmutador **Configurar** en **Sí**, asegúrese de que las casillas **Explorador** y **Aplicaciones móviles y clientes de escritorio** estén activadas y haga clic en **Listo**.
   - En la sección **Controles de acceso**, haga clic en **Conceder**; en la hoja **Conceder**, asegúrese de que la opción **Conceder acceso** está seleccionada, active la casilla **Requerir autenticación multifactor** y haga clic en **Seleccionar**.
   - En la sección **Controles de acceso**, haga clic en **Sesión**; en la hoja **Sesión**, active la casilla **Frecuencia de inicio de sesión**; en el primer cuadro de texto, escriba **4**; en la lista desplegable **Seleccionar unidades**, seleccione **Horas**; deje desactivada la casilla **Sesión del explorador persistente** y haga clic en **Seleccionar**.
   - Establezca el conmutador **Habilitar directiva** en **Activado**.

1. En la hoja **Nuevo**, haga clic en **Crear**. 

#### <a name="task-2-test-the-azure-ad-based-conditional-access-policy-for-all-azure-virtual-desktop-connections"></a>Tarea 2: Probar la directiva de acceso condicional basado en Azure AD de todas las conexiones de Azure Virtual Desktop

1. En el equipo de laboratorio, en la ventana del explorador web donde se muestra Azure Portal, abra la sesión del shell de **PowerShell** en el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para iniciar las máquinas virtuales de Azure del host de sesión de Azure Virtual Desktop que vamos a usar en este laboratorio:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Nota**: Espere a que se complete el comando y se ejecuten todas las máquinas virtuales de Azure del grupo de recursos **az140-21-RG**. 

1. En el equipo de laboratorio, abra una sesión de explorador web **InPrivate**, vaya a [Azure Portal](https://portal.azure.com) e inicie sesión indicando el nombre principal de usuario de **aduser5** que localizamos anteriormente en este ejercicio y la contraseña que establecimos al crear esta cuenta de usuario.

   > **Nota**: Confirme que no se le pide autenticarse a través de MFA.

1. En la sesión del explorador web **InPrivate**, vaya a la página de cliente web HTML5 de Azure Virtual Desktop en [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Nota**: Confirme que esto hace que se desencadene automáticamente la autenticación mediante MFA.

1. En el panel **Escribe el código**, escriba el código del mensaje de texto o la aplicación autenticadora que registramos y seleccione **Comprobar**.
1. En la página **Todos los recursos**, haga clic en **Símbolo del sistema** y, en el panel **Access local resources** (Acceder a recursos locales), desactive la casilla **Impresora** y haga clic en **Permitir**.
1. Cuando se le solicite, escriba el nombre principal de usuario de **aduser5** en el cuadro de texto **Nombre de usuario** de la sección **Escriba sus credenciales** y, en el cuadro de texto **Contraseña**, escriba la contraseña que establecimos al crear esta cuenta de usuario y haga clic en **Enviar**.
1. Compruebe que la aplicación remota **Símbolo del sistema** se ha iniciado correctamente.
1. En la ventana de la aplicación remota **Símbolo del sistema**, en el símbolo del sistema, escriba **logoff** y presione la tecla **Entrar**.
1. De nuevo en la página **Todos los recursos**, en la esquina superior derecha, haga clic en **aduser5**, en el menú desplegable, haga clic en **Cerrar sesión** y cierre la ventana del explorador web **InPrivate**.

#### <a name="task-3-modify-the-azure-ad-based-conditional-access-policy-to-exclude-hybrid-azure-ad-joined-computers-from-the-mfa-requirement"></a>Tarea 3: Modificar la directiva de acceso condicional basado en Azure AD para excluir los equipos unidos a Azure AD híbrido del requisito de MFA

>**Nota**: En esta tarea, modificaremos la directiva de acceso condicional basado en Azure AD que requiere MFA para iniciar sesión en una sesión de Azure Virtual Desktop, de modo que las conexiones que provengan de equipos unidos a Azure AD no requerirán MFA.

1. En el equipo de laboratorio, en la ventana del explorador donde se muestra Azure Portal, en la hoja **Acceso condicional | Directivas**, haga clic en la entrada de la directiva **az140-31-wvdpolicy1**.
1. En la hoja **az140-31-wvdpolicy1**, en la sección **Controles de acceso**, haga clic en **Conceder**; en la hoja **Conceder**, active las casillas **Requerir autenticación multifactor** y **Requerir dispositivo unido a Azure AD híbrido**, asegúrese de que la opción **Requerir uno de los controles seleccionados** está habilitada y haga clic en **Seleccionar**.
1. En la hoja **az140-31-wvdpolicy1**, haga clic en **Guardar**.

>**Nota**: La directiva puede tardar unos minutos en surtir efecto.

#### <a name="task-4-test-the-modified-azure-ad-based-conditional-access-policy"></a>Tarea 4: Probar la directiva de acceso condicional basado en Azure AD modificado

1. En el equipo de laboratorio, en la ventana del explorador donde se muestra Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az140-cl-vm11**.
1. En la hoja **az140-cl-vm11**, seleccione **Conectar**; en el menú desplegable, seleccione **Bastion**; en la pestaña **Bastion** de la hoja **az140-cl-vm11 \| Conectar**, seleccione **Usar Bastion**.
1. Cuando se le solicite, proporcione las credenciales siguientes y seleccione **Conectar**:

   |Configuración|Value|
   |---|---|
   |Nombre de usuario|**Student@adatum.com**|
   |Contraseña|**Pa55w.rd1234**|

1. En la sesión de Escritorio remoto a **az140-cl-vm11**, inicie Microsoft Edge y vaya a la página del cliente web HTML5 de Azure Virtual Desktop en [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Nota**: Confirme que esta vez no se le pide que se autentique a través de MFA. Esto se debe a que **az140-cl-vm11** está unido a Azure AD híbrido.

1. En la página **Todos los recursos**, haga doble clic en **Símbolo del sistema** y, en el panel **Access local resources** (Acceder a recursos locales), desactive la casilla **Impresora** y haga clic en **Permitir**.
1. Cuando se le solicite, escriba el nombre principal de usuario de **aduser5** en el cuadro de texto **Nombre de usuario** de la sección **Escriba sus credenciales** y, en el cuadro de texto **Contraseña**, escriba la contraseña que establecimos al crear esta cuenta de usuario y haga clic en **Enviar**.
1. Compruebe que la aplicación remota **Símbolo del sistema** se ha iniciado correctamente.
1. En la ventana de la aplicación remota **Símbolo del sistema**, en el símbolo del sistema, escriba **logoff** y presione la tecla **Entrar**.
1. De nuevo en la página **Todos los recursos**, en la esquina superior derecha, haga clic en **aduser5** y, en el menú desplegable, haga clic en **Cerrar sesión**.
1. En la sesión de Escritorio remoto a **az140-cl-vm11**, haga clic en **Inicio**; en la barra vertical justo encima del botón **Inicio**, haga clic en el icono que representa la cuenta de usuario que ha iniciado sesión y, en el menú emergente, haga clic en **Cerrar sesión**.

### <a name="exercise-3-stop-and-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>Ejercicio 3: Detener y desasignar las máquinas virtuales de Azure aprovisionadas en el laboratorio

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
