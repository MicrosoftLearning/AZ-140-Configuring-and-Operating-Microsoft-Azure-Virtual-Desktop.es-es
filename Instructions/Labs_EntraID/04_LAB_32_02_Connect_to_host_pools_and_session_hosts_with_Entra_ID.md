---
lab:
  title: 'Laboratorio: Conexión a hosts de sesión (Entra ID)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Laboratorio: Conexión a hosts de sesión (Entra ID)
# Manual de laboratorio para estudiantes

## Dependencias de laboratorio

- Una suscripción a Azure que usarás en este laboratorio.
- Una cuenta de usuario de Microsoft Entra con los roles de Propietario o Colaborador en la suscripción a Azure que usarás en este laboratorio y con los permisos suficientes para unir dispositivos al inquilino de Microsoft Entra asociado a esa suscripción a Azure.
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*
- Haber completado el laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)*

## Tiempo estimado

20 minutos

## Escenario del laboratorio

Tienes un entorno de Azure Virtual Desktop existente que contiene hosts de sesión unidos a Entra. Para validar su funcionalidad, conéctate a ellas desde un cliente de Windows 11 que no esté unido a Microsoft Entra ni registrado.

## Objetivos
  
Después de completar este laboratorio, podrás:

- Validar la funcionalidad de los hosts de sesión de Azure Virtual Desktop unidos a Microsoft Entra mediante la conexión desde un cliente de Windows que no esté unido a Microsoft Entra ni registrado.

## Archivos de laboratorio

- Ninguno

## Instrucciones

### Ejercicio 1: Validación de la funcionalidad de los hosts de sesión de Azure Virtual Desktop unidos a Microsoft Entra mediante la conexión desde un cliente de Windows 11
  
Las tareas principales de este ejercicio son las siguientes:

1. Ajuste de las propiedades de RDP del grupo de hosts de Azure Virtual Desktop
1. Instalación del cliente Escritorio remoto de Microsoft en un equipo con Windows 11
1. Suscripción a un área de trabajo de Azure Virtual Desktop
1. Prueba de aplicaciones de Azure Virtual Desktop


#### Tarea 1: Ajuste de las propiedades de RDP del grupo de hosts de Azure Virtual Desktop

> **Nota**: la configuración de RDP que implementaste en el laboratorio anterior proporciona la experiencia de usuario óptima (a través de la compatibilidad con el inicio de sesión único), pero esto requiere los cambios adicionales descritos en [Configuración del inicio de sesión único para Azure Virtual Desktop mediante la autenticación de Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on). Sin estos cambios, de forma predeterminada, se admite la autenticación, siempre que el equipo cliente cumpla uno de los siguientes criterios:

- Está unido a Microsoft Entra en el mismo inquilino de Microsoft Entra que el host de sesión
- Está unido híbrido a Microsoft Entra en el mismo inquilino de Microsoft Entra que el host de sesión
- Está registrado en Microsoft Entra en el mismo inquilino de Microsoft Entra que el host de sesión

Dado que ninguno de estos criterios se aplica al equipo de laboratorio, es necesario agregar `targetisaadjoined:i:1` como una propiedad de RDP personalizada al grupo de hosts.

1. De ser necesario, en el equipo de laboratorio, inicia un explorador web, ve a Azure Portal e inicia sesión con las credenciales de una cuenta de usuario con el rol de propietario en la suscripción que usarás en este laboratorio.

    > **Nota**: usa las credenciales de la cuenta `User1-` que aparecen en la pestaña Recursos del lado derecho de la ventana de sesión del laboratorio.

1. En el explorador web que muestra Azure Portal, en la página **az140-21-hp1**, en la barra de menús vertical, en la sección **Configuración**, selecciona la entrada **Propiedades de RDP**.
1. En la página **az140-21-hp1 \| Propiedades de RDP**, selecciona la pestaña **Avanzado**. 
1. En la pestaña **Avanzado** de la página **az140-21-hp1 \| Propiedades de RDP**, en el cuadro de texto **Propiedades de RDP**, anexa la siguiente cadena al contenido existente (asegúrate de agregar un carácter de punto y coma inicial (`;`) si es necesario para separar esta cadena del que lo preceda:

    ```txt
    targetisaadjoined:i:1
    ```

1. En el cuadro de texto **Propiedades de RDP**, quita la siguiente cadena (si está presente) del contenido existente (con su carácter de punto y coma final):

    ```txt
    enablerdsaadauth:i:value
    ```

1. En la página **az140-21-hp1 \| Propiedades de RDP**, selecciona **Guardar**.

#### Tarea 2: Instalación del cliente Escritorio remoto de Microsoft en un equipo con Windows 11

1. En el equipo de laboratorio, inicia un explorador web, ve a la página [Conectar a Azure Virtual Desktop con el cliente de Escritorio remoto para Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), desplázate hacia abajo hasta la sección **Descargar e instalar el cliente de Escritorio remoto (MSI)** y selecciona el vínculo de [Windows de 64 bits](https://go.microsoft.com/fwlink/?linkid=2139369). 
1. Abre Explorador de archivos, ve a la carpeta **Descargas** e inicia la instalación del archivo MSI recién descargado. 
1. En la ventana **Configuración de Escritorio remoto**, cuando se te solicite, acepta los términos del contrato de licencia y elige la opción **Instalar para todos los usuarios de esta máquina**. Si se te solicita, acepta el menaje del control de cuentas de usuario para continuar con la instalación.
1. Una vez completada la instalación, asegúrate de que la casilla **Iniciar Escritorio remoto cuando se cierre la instalación** esté activada y haz clic en **Finalizar** para iniciar el cliente de Escritorio remoto de Microsoft.

   > **Nota**: la [aplicación Tienda de Escritorio remoto](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store) para Windows no admite la conexión a hosts de sesión unidos a Microsoft Entra.

#### Tarea 3: Suscripción a un área de trabajo de Azure Virtual Desktop

1. En el equipo de laboratorio, cambia a la ventana del cliente de **Escritorio remoto**, selecciona **Suscribirse** y, cuando se te solicite, inicia sesión con las credenciales de la `User1` cuenta de usuario de Entra ID que puedes encontrar en la pestaña **Recursos** del panel derecho de la ventana de la interfaz del laboratorio.

   > **Nota**: selecciona la cuenta de usuario que es miembro del grupo Entra con el prefijo **AVD-DAG**.

   > **Nota**: como alternativa, en la ventana cliente de **Escritorio remoto**, selecciona **Suscribirse con dirección URL**, en el panel **Suscribirse a un área de trabajo**, en la **dirección URL de correo electrónico o área de trabajo**, escribe **https://client.wvd.microsoft.com/api/arm/feeddiscovery**, selecciona **Siguiente** y, una vez que se te solicite, inicia sesión con las credenciales de Microsoft Entra.

1. Asegúrate de que la página **Escritorio remoto** muestra solo el icono **SessionDesktop**.

   > **Nota**: esto es previsible, ya que la cuenta de usuario de Microsoft Entra seleccionada se asignó en el primer laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)* al grupo de aplicaciones de escritorio **az140-21-hp1-DAG** generado automáticamente.

1. En la página **Escritorio remoto**, haz clic con el botón derecho en el icono **SessionDesktop** y, en el menú emergente, selecciona **Configuración**.
1. En el panel **SessionDesktop**, desactiva el modificador **Usar configuración predeterminada**.
1. En la sección **Configuración de pantalla**, en el menú desplegable, selecciona la entrada **Seleccionar pantallas** y elige las pantallas que deseas usar para la sesión.
1. En el panel **SessionDesktop**, revisa las opciones restantes, incluyendo **Maximizar a las pantallas actuales**, **Pantalla única cuando está en modo ventana** y **Ajustar sesión a ventana**, sin realizar ningún cambio. 
1. Cierra el panel **SessionDesktop**. 
1. En la página **Escritorio remoto**, haz doble clic en el icono **SessionDesktop**.
1. Cuando se te pida que inicies sesión, en el cuadro de diálogo **Seguridad de Windows**, escribe la contraseña de la primera cuenta de usuario de Microsoft Entra, que usaste en esta tarea para conectarte al entorno de Azure Virtual Desktop de destino.

   > **Nota**: Azure Virtual Desktop no admite el inicio de sesión en Microsoft Entra ID con una cuenta de usuario y después el inicio de sesión en Windows con otra cuenta de usuario. Iniciar sesión con dos cuentas diferentes al mismo tiempo puede provocar que los usuarios se vuelvan a conectar con el host de sesión incorrecto, que falte información en el Azure Portal o que la información sea incorrecta, y que aparezcan mensajes de error al usar la conexión de aplicaciones o la conexión de aplicaciones MSIX.

   > **Nota**: se te presentará automáticamente la ventana **SessionDesktop**.

1. En la ventana de sesión de Escritorio remoto, comprueba que tienes acceso administrativo completo dentro de la sesión (por ejemplo, seleccione el icono de logotipo de **Windows** en la barra de tareas y, a continuación, selecciona el elemento **Windows PowerShell(Admin)** en el menú emergente.
1. En la ventana de sesión de Escritorio remoto, selecciona el icono de logotipo de Windows en la barra de tareas, selecciona el icono de avatar que representa la cuenta de usuario de Microsoft Entra que usaste para iniciar sesión y, en el menú emergente, selecciona **Cerrar sesión**.

   > **Nota**: esto finalizará automáticamente la sesión de Escritorio remoto. 

1. De nuevo en la ventana **Escritorio remoto**, selecciona el icono de puntos suspensivos (`...`) situado a la derecha de la entrada del área de trabajo **az140-21-ws1**, selecciona **Cancelar suscripción** y, cuando se te pida que confirmes, selecciona **Continuar**.
1. En la ventana cliente de **Escritorio remoto**, selecciona **Suscribirse** y, cuando se te solicite, inicia sesión con las credenciales de la segunda cuenta de usuario de Entra ID que encontrarás en la pestaña **Recursos** del panel derecho de la ventana de la interfaz de laboratorio.

   > **Nota**: selecciona la cuenta de usuario que es miembro del grupo Entra con el prefijo **AVD-RemoteApp**.

1. Asegúrate de que en la página **Escritorio remoto** se muestran cuatro iconos, incluidos el símbolo del sistema, Microsoft Word, Microsoft Excel y Microsoft PowerPoint. 

   > **Nota**: esto es previsible, ya que la cuenta de usuario de Microsoft Entra que seleccionaste se asignó en el primer laboratorio *Implementación de grupos de hosts y hosts de sesión mediante Azure Portal (Entra ID)* en los grupos de aplicaciones **az140-21-hp1-Office365-RAG** y **az140-21-hp1-Utilities-RAG**.

1. Haz doble clic en el icono del símbolo del sistema. 
1. Cuando se te pida que inicies sesión, en el cuadro de diálogo **Seguridad de Windows**, escribe la contraseña de la segunda cuenta de usuario de Microsoft Entra que usaste para conectarte al entorno de Azure Virtual Desktop de destino.
1. Comprueba que aparece una ventana del **símbolo del sistema** poco después. 
1. En la ventana del símbolo del sistema, escribe **hostname** y presiona la tecla **Intro** para mostrar el nombre del equipo en el que se ejecuta el símbolo del sistema.

   > **Nota**: comprueba que el nombre mostrado comienza con el prefijo **sh-**.

1. En el símbolo del sistema, escribe **logoff** y presiona la tecla **Intro** para cerrar sesión desde la sesión de aplicación remota actual.
1. Haz doble clic en los iconos restantes de la página **Escritorio remoto** para iniciar Microsoft Word, Microsoft Excel y Microsoft PowerPoint.
1. Cierra cada ventana de sesión.