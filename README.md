# DOCUMENTACIÓN SNIPE-IT
En este manual se cubrirá la información básica para la implementación con Docker y uso de Snipe-it.

## Índice<a name="inicio"></a>
- [¿Qué es Snipe-it y qué ofrece?](#definicion)
- [Implementación con Docker al servidor de pruebas Sofía](#implementacion)
  - [Descarga de Snipe-it](#descarga)
  - [Configuración de las variables de entorno](#env)
  - [Parámetros de MySQL](#parametros)
  - [Opciones de Snipe-it](#opciones)
  - [Creación e inicio del contenedor con MYSQL 5.6](#mysql)
- [Inicio del contenedor Snipe-it](#snipeit)
- [Página de set-up](#pagina_inicio)
- [Pasos básicos para agregar modelos de equipos, categorías, fabricantes, proveedores, departamentos, localizaciones, etc.](#modelos)
- [Pasos básicos para agregar equipos, licencias, consumibles, etc.](#equipos)
- [Exportar bases de datos](#exportar)
- [Importar bases de datos](#importar)
- [Registro de mantenimiento de equipos y más](#registros)
- [Obtención de informes](#informes)

## ¿Qué es Snipe-it y qué ofrece?<a name="definicion"></a>
![snipeit_logo.png](imagenes/snipeit_logo.png)
<div class=text-justify>
Snipe-it, en pocas palabras, es un administrador de recursos (licencias, equipos, consumibles, componentes y accesorios) de licencia abierta en formato de aplicación web. La manera de permitir agregar, quitar, editar y ver los recursos hace de Snipe-it una de las aplicaciones de manejo de recursos más usadas por empresas y escuelas.
  
Las bondades que ofrece Snipe-it son las siguientes:
* Administrar equipos, accesorios, licencias y consumibles.
* Controlar fechas de expiración de liciencias, mantenimiento de equipos, accesorios y componentes y fechas de compra de recursos.
* Tener registro de las empresas a las que se les adiquiró recursos.
* Poder consultar en todo momento el número de serie del equipo, licencia o accesorio en cuestión, en el caso de la licencia la clave de activación y el equipo donde fue aplicada.
* Marcar los recursos como almacenados, pendientes de entrega y listos para usar.
* Identificar con facilidad los recursos mediante el uso de imágenes.
* Importar archivos CSV.
* Crear reportes de actividad, auditoría, amortización, licencias, mantenimiento de equipos.
* Tener un control de calidad sobre los activos, por medio de no conformidades.
* Tener flexibilidad en el reporte de equipos.
* Registrar el préstamo de equipos, así como las fechas esperadas de devolución, su estado y la localización esperada de ellos.
* Asignación de recursos.
</div>

[Regresar al índice](#inicio)

## Implementación con Docker al servidor de pruebas Sofía <a name="implementacion"></a>
La implementación de Snipe-it a cualquier servidor (en este caso Sofía) por medio de Docker, se requiere seguir una serie de pasos, los cuáles a continuación se mostrarán:

### Descarga de Snipe-it <a name="descarga"></a>
                                           
Para descarga del hub de Docker la imagen más reciente de Snipe-it en Docker se usa el siguiente comando:
<br>`docker pull snipe/snipe-it`</br>
Y para comprobar que efectivamente se completó la descarga y esté disponible en Docker se usa el siguiente comando:
<br>`docker images`</br>
El cuál al ejecutarlo nos mostrará todas las imágenes disponibles, siendo la de nombre snipe/snipe-it la de mayor interés por este momento:
![docker_images](imagenes/docker_images.png)

### Configuración de las variables de entorno <a name="env"></a>
Snipe-it requiere de la creación de un archivo```.env```para poder funcionar. Para ello vamos a crear un archivo de extensión```.env```con el siguiente comando:

```
nano nombre_del_archivo.env
```

Se desplegará en la terminal el siguiente editor de texto:
![nano_vacio](imagenes/nano_vacio.png)

En el cuál pegaremos lo siguiente:

```
# Mysql Parameters
MYSQL_ROOT_PASSWORD=YOUR_SUPER_SECRET_PASSWORD
MYSQL_DATABASE=snipeit
MYSQL_USER=snipeit
MYSQL_PASSWORD=YOUR_snipeit_USER_PASSWORD

# Email Parameters
# - the hostname/IP address of your mailserver
MAIL_PORT_587_TCP_ADDR=smtp.whatever.com
#the port for the mailserver (probably 587, could be another)
MAIL_PORT_587_TCP_PORT=587
# the default from address, and from name for emails
MAIL_ENV_FROM_ADDR=youremail@yourdomain.com
MAIL_ENV_FROM_NAME=Your Full Email Name
# - pick 'tls' for SMTP-over-SSL, 'tcp' for unencrypted
MAIL_ENV_ENCRYPTION=tcp
# SMTP username and password
MAIL_ENV_USERNAME=your_email_username
MAIL_ENV_PASSWORD=your_email_password

# Snipe-IT Settings
APP_ENV=production
APP_DEBUG=false
APP_KEY=<<Fill in Later!>>
APP_URL=http://127.0.0.1:YOUR_PORT_NUMBER
APP_TIMEZONE=US/Pacific
APP_LOCALE=en
  
# Docker-specific variables
PHP_UPLOAD_LIMIT=100
```

#### Parámetros de MySQL<a name="parametros"></a>
Para configurar los parámetros de MySQL vamos a entender primero cada uno de ellos:
````
MYSQL_ROOT_PASSWORD=
````
Este parámetro indica la contraseña que va a pedir MySQL cuando se quiera acceder al contenedor del mismo por medio de la consola de comandos para modificar las bases de datos, permisos, usuarios, etc.

```
MYSQL_DATABASE=
```
Es el nombre de la base de datos de MySQL que Snipe-it va a crear y usar. Es recomendable dejarla con el valor de `snipeit`.

```
MYSQL_USER=
````
Es el usuario por default a crear en MySQL.

```
MYSQL_PASSWORD=
```
Es la contraseña del usuario creado anteriormente.

#### Opciones de Snipe-it<a name="opciones"></a>
En este apartado vamos a cambiar lo siguiente:
```
APP_URL=http://127.0.0.1:YOUR_PORT_NUMBER
```
En lugar de `127.0.0.1`colocaremos la dirección ip del servidor de pruebas Sofía, el cuál al día 17 de febrero de 2023 es `192.168.119.69`y cómo puerto se pondrá uno desponible (preguntar a los encargados de servicio social sobre qué puertos usar y cuáles no).

`APP_TIMEZONE=US/Pacific`y`APP_LOCALE`especifican la zona horaria y el idioma de la aplicación respectivamente. Para la Ciudad de México la zona horaria es `America/Mexico_City` y el idioma es `es-ES`

¡Y listo! Ya tenemos configuradas las variables de entorno de nuestra aplicación, por lo que podemos empezar a correr el contenedor MySQL, procedimiento que se verá a continuación.

### Creación e inicio del contenedor con MYSQL 5.6<a name="mysql"></a>
El comando para iniciar el contenedor MySQL es el siguiente:
```
docker run --name snipe-mysql --env-file=my_env_file --mount source=snipesql-vol,target=/var/lib/mysql -d -P mysql:5.6
```
Donde: 
* `--name` indica el nombre del contenedor.
* `--env-file`es el nombre del archivo.
* `.env` que creamos en un principio.
* `snipesql-vol` es el volumen de Snipe-it que se crea para MySQL.
* `-d` es la indicación para ocultar el log del comando que se ejecutó.
* `mysql:` es la versión de MySQL a usar.

>ATENCIÓN: Es importante usar la versión de MySQL menor o igual a 5.6, pues si se usa una mayor Snipe-it no va a conectarse con MySQL debido a que estas corren en modo estricto por default.


Una vez puesto en marcha el comando, la terminal debería de mostrar lo siguiente:

![mysql_start](imagenes/mysql_start.png)

Los números que entrega la terminal es el ID del contendor. Para revisar si está en linea usamos el comando `docker ps` y debería de aparecer la ID que vimos anteriormente con el nombre del contenedor que pusimos en el comando de creación del contenedor MYSQL, en este caso: `snipe-mysql`.

![docker_ps_1](imagenes/docker_ps_1.png)

Cómo se muestra en la terminal, quiere decir que nuestra base de datos ya se encuentra en línea en el servidor y lista para ser usada por Snipe-it.

[Regresar al índice](#inicio)

## Inicio del contenedor Snipe-it<a name="snipeit"></a>
Iniciar el contenedor Snipe-it requiere de la generación de un APP_KEY, el cuál se va a generar mediante el siguiente comando:
`docker run --rm snipe/snipe-it`

No nos va a dejar iniciar el contenedor, pues nos dice que hace falta generar un APP_KEY para usar en el archivo .env, seguido de un ejemplo de APP_KEY que perfectamente podemos usar:

![app_key](imagenes/app_key.png)

Vamos a copiar la clave que generó la terminal y la vamos a pegar en el apartado `APP_KEY=` que está dentro del archivo `.env` que creamos y editamos hace unos momentos:

![app_key_env](imagenes/app_key_env.png)

Una vez hecho esto, podemos correr nuestro contenedor Snipe-it con el siguiente comando:
```
docker run -d -p YOUR_PORT_NUMBER:80 --name="snipeit" --link snipe-mysql:mysql --env-file=my_env_file --mount source=snipe-vol,dst=/var/lib/snipeit snipe/snipe-it
```

donde:
* `YOUR_PORT_NUMBER` es el puerto que previamente colocamos en el archivo `.env`.
* `snipeit` es el nombre que le vamos a poner al contenedor
* `--link snipe-mysql:mysql` es la indicación de que vamos a conectar el contenedor de Snipe-it con el contenedor de MySQL que creamos anteriormente.
* `--env-file`es el archivo `.env` que creamos previamente.

De tal suerte que si lo editamos correctamente, vamos a poder ver lo siguiente en la terminal:

![snipeit_start](imagenes/snipeit_start.png)

Y al usar nuevamente el comando `docker ps`apreciaremos que está en línea el contenedor Snipe-it:

![docker_ps_2](imagenes/docker_ps_2.png)

[Regresar al índice](#inicio)

## Página de set-up<a name="pagina_inicio"></a>
Si hicimos todo bien, al ingresar a la dirección que pusimos en nuestro archivo `.env` desde nuestro navegador preferido, podremos ver el pre-flight de Snipe-it, es decir, la configuración inicial de la app en si de manera mucho más intuitiva.

![snipeit_web](imagenes/snipeit_web.png)

Aquí nos indicará Snipe-it si hay algún problema con la configuración inicial, se mostrará de color rojo junto con la descripción del problema. Para consultar los problemas más comunes con su solución podemos visitar el FAQ de Snipe-it con la siguiente liga:
<a href="https://snipe-it.readme.io/docs/common-issues">Problema comunes con la instalación de Snipe-it</a>

Al continuar con los siguientes pasos en la página de Snipe-it, llegaremos a una página donde tenemos que configurar aspectos básicos de la aplicación, cómo lo son:
* Nombre del sitio de Snipe-it, por ejemplo: Adminisitración de recursos USECAD.
* Idioma de la app.
* Moneda.
* Extensión de los ID's de los recursos.

Así como el dominio de los correos que se van a usar en la aplicación y el estilo de los correos a partir de los nombres de las personas. También pedirá nombre y apellido para el perfil del usuario inicial, su correo, usuario a usar dentro de Snipe-it y una contraseña para ingresara a dicho perfil.

![snipeit_config](imagenes/snipeit_config.png)

Al terminar con la configuración, puede aparecer `500 web server error`, no pasa nada, solo basta con entrar de nuevo a la dirección ip y debería de aparecer la página de inicio de Snipe-it.

![snipeit_home](imagenes/snipeit_home.png)

[Regresar al índice](#inicio)

## Pasos básicos para agregar modelos de equipos, categorías, fabricantes, proveedores, departamentos, localizaciones, etc.<a name="modelos"></a>
La simplicidad y versatilidad de Snipe-it como una base de datos con interfaz gráfica permite que sea muy fácil identificar los pasos para agregar activos, de hecho, la información que se pida para rellenar va a variar dependiendo del activo que se esté agregando.
1. Sobre la barra de opciones de la página inicio hacer click sobre la barra y seleccionar el engrane de opciones.
2. Dar click en "Modelos".
3. Seleccionar "Crear localización".
4. Aparecerá la siguiente página con información a llenar:

![config_modelos](imagenes/config_modelos.png)

Aquí podremos agregar el fabricante, la categoría en la que está (puede ser laptop, PC, impresora, etc. el límite es tu imaginación), el número de modelo e incluso la depreciación del equipo si es que se le va a tomar en cuenta (la amortización se explicará más tarde).
5. Podremos agregar una imagen para identificar al equipo rápidamente.
6. Finalmente daremos click en guardar.
7. El modelo aparecerá en la pantalla junto con otra información.

Así como se agregaron modelos, se pueden agregar departamentos, localizaciones, fabricantes, proveedores, etc., todo lo que esté dentro del apartado de opciones.

>Se deben de agregar los modelos primero para tener una base de todo lo que se tiene en el lugar de trabajo, para que a la hora de agregar equipos se puedan seleccionar fácilmente, agregar el número de unidades que se tienen, sus números de serie, etc.

[Regresar al índice](#inicio)

## Pasos básicos para agregar equipos, licencias, consumibles, etc.<a name="equipos"></a>
Al igual que los pasos básicos 
Para agregar equipos se tiene que hacer lo siguiente:
1. En la página de inicio dar click derecho sobre las barra laterar para desplegar las opciones
2. Elegir "Equipos" seguido de "Listar todo".
3. Dar click en "Crear localización" y se desplegará un formulario donde agregaremos lo que pida.

![snipeit_equipos](imagenes/snipeit_equipos.png)

4. Podremos notar que podemos seleccionar modelos que previamente creamos, darle un nombre al equipo para ser fácilmente identificado, así cómo una etiqueta de identificación y el estado en el que se encuentra (no estado de México, sino que si está en el almacén, pendiente de preparar o listo para usar). También podremos agregar información adicional del equipo en cuestión, cómo la fecha en el que se compró, el costo que tuvo, el número de pedido y el proveedor de dicho activo, información que seguro nos vendrá bien en un futuro.
5. Daremos click en agregar y ¡listo!.

Recordemos que estos pasos son similares con cada uno de los activos que snipe-it permite agregar.

[Regresar al índice](#inicio)

## Exportar bases de datos<a name="exportar"></a>
Otra de las bondades que ofrece Snipe-it es la exportación de las bases de datos que nosostros creamos de cualquier activo en la extensión de archivo que mejor nos convenga. Para ello vamos a ir a cualquiera de las categorías de Snipe-it y observaremos que en la parte superior derecha hay un botón de descarga. 

![desgarga_datos](imagenes/descarga_datos.png)

El cuál a hacer click nos desplegará una lista de todas las extensiones en la que podemos descargar el archivo, las cuáles son:
* MS-Excel (Open XML)
* MS-Excel
* CSV
* PDF
* JSON
* SML
* TXT
* SQL
* MS-Word
Al seleccionar cualquiera de las opciones posibles, automáticamente se descargar un archivo que de nombre tiene la palabra "export" y la categoría de activo que estamos exportando, así como la fecha en la que se exportó dicho archivo. La información que tendrá dicho archivo es todo lo que tengamos de dicha categoría y está se mostrará en el formato en el que la hayamos descargado.

[Regresar al índice](#inicio)

## Importar bases de datos<a name="importar"></a>
Snipe-it permite importar bases de datos únicamente en formato CSV. Para importar vamos a la barra de la parte izquierda de nuestra interfaz y seleccionaremos la opción "Importar".

![importar_snipeit](imagenes/importar_snipeit.png)

Al seleccionar dicha opción se desplegará una pantalla donde tendremos que subir el archivo `.csv`. Una vez subido a Snipe-it, podremos elegir más opciones de cómo queremos que se clasifiquen los datos que contiene nuestro `.csv` con tan solo dar click en el recuadro azul, una pantalla similar a esta se debería de desplegar.

![clasificar_snipeit](imagenes/clasificar_snipeit.png)

Cuándo terminar de clasificar nuestros datos, podremos ir al apartado del tipo de activos que subimos a Snipe-it para comprobar que efectivamente se importaron.

![verificar_importacion](imagenes/verificar_importacion.png)

>Es importante seguir el formato CSV que Snipe-it pide para la lectura correcta de datos por parte de la aplicación. Para más información de este formato vista: <a href="https://snipe-it.readme.io/docs/importing">Importando activos y más</a>.

[Regresar al índice](#inicio)

## Registro de mantenimiento de equipos y más<a name="registros"></a>
Snipe-it permite tener un regsitro de lo siguiente:
* Prestaciones de equipos a usuarios.
* Mantenimiento a equipos.
* Equipos solicitados por otros usarios.
* Equipos borrados.
* Historial de importaciones de datos.
* Fechas de auditorás masivas.
Para acceder a ellos, debemos de dar click sobre la misma barra de siempre de nuestro lado izquierdo y seleccionar la opción "Equipos", en la cuál se desplegará una gran cantidad de opciones, entre ellas las que mencionamos anteriormente, las cuáles estarán hasta abajo del listado. El dar click sobre cualquiera de otras opciones desplegará formularios similares a los antes vistos, por supuesto, con diferentes datos a pedir.

Un ejemplo de esto es la siguiente imagen del formulario que se abre al seleccionar la opción de mantenimiento de equipos:

![mantenimiento_snipeit](imagenes/mantenimiento_equipos.png)

[Regresar al índice](#inicio)

## Obtención de informes<a name="informes"></a>
Snipe-it permite generar informes y reportes de lo siguiente:
* Actividad de los usarios
* Auditorías
* Devaluaciones
* Licencias
* Mantenimientos realizados
* No conformidades con equipos
* Accesorios

Para acceder a ellos bastará con ir a la siempre confiable barra lateral izquierda y navegar por la opciones hasta el fondo para encontrar la opción "Informes", el cuál al dar click desplegará las opciones anteriormente mencionadas:

![informes_snipeit](imagenes/informes_snipeit.png)

Como siempre al dar click sobre cualquier opción disponible desplegará una ventana pero esta vez con información de dicha categoría. Por ejemplo, con la opción de mantenimiento podremos ver información de cuándo fué el último servicio, fecha de del siguiente,etc. y toda esta información la podremos exportar en el formato de archivo que mejor nos convenga.

![informe_ejemplo](imagenes/informe_ejemplo.png)

¡Y listo! esta es toda la información básica que se necesita saber sobre Snipe-it para crear registros de activos, lo que nos va a permitir tener un mejor control sobre todo lo que tenemos en nuestra empresa o centro de trabajo, como lo es un centro de base de datos de la facultad de Ingeniería donde hay equipos, componentes, accesorios y licencias por doquier.

[Regresar al índice](#inicio)
