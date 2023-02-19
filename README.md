# DOCUMENTACIÓN SNIPE-IT
En este manual se cubrirá la información básica para la implementación con Docker y uso de Snipe-it.

## Índice
- [¿Qué es Snipe-it y qué ofrece?](#)
- [Implementación con Docker al servidor de pruebas Sofía](#implementación_con_docker_al_servidor_de_pruebas_sofía)
- [Descarga de Snipe-it](#descarga_de_snipe-it)
- [Configuración de las variables de entorno](#configuración_de_las_variables_de_entorno)
- [Parámetros de MySQL](#parámetros_de_mysql)
- [Opciones de Snipe-it](#opciones_de_snipe-it)
- [Creación e inicio del contenedor con MYSQL 5.6](#creación_e_inicio_del_contenedor_con_MYSQL_5.6)
- [Inicio del contenedor Snipe-it](#inicio_del_contenedor_snipeit)
- [Resultado final](#resultado_final)

## ¿Qué es Snipe-it y qué ofrece?<a name="definicion"></a>
![snipeit_logo.png](imagenes/snipeit_logo.png)
<p style="text-align": justify;">Snipe-it, en pocas palabras, es un administrador de recursos (licencias, equipos, consumibles, componentes y accesorios) de licencia abierta en formato de aplicación web. La manera de permitir agregar, quitar, editar y ver los recursos hace de Snipe-it una de las aplicaciones de manejo de recursos más usadas por empresas y escuelas.</p> 

<p style="text-align": justify;">
Las bondades que ofrece Snipe-it son las siguientes:
*Administrar equipos, accesorios, licencias y consumibles.
*Controlar fechas de expiración de liciencias, mantenimiento de equipos, accesorios y componentes y fechas de compra de recursos.
*Tener registro de las empresas a las que se les adiquiró recursos.
*Poder consultar en todo momento el número de serie del equipo, licencia o accesorio en cuestión, en el caso de la licencia la clave de activación y el equipo donde fue aplicada.
*Marcar los recursos como almacenados, pendientes de entrega y listos para usar.
*Identificar con facilidad los recursos mediante el uso de imágenes.
*Importar archivos CSV.
*Crear reportes de actividad, auditoría, amortización, licencias, mantenimiento de equipos.
*Tener un control de calidad sobre los activos, por medio de no conformidades.
*Tener flexibilidad en el reporte de equipos.
*Registrar el préstamo de equipos, así como las fechas esperadas de devolución, su estado y la localización esperada de ellos.
*Asignación de recursos.</p>
[Inicio](#índice)
## Implementación con Docker al servidor de pruebas Sofía
<p style="text-align": justify;">La implementación de Snipe-it a cualquier servidor (en este caso Sofía) por medio de Docker, se requiere seguir una serie de pasos, los cuáles a continuación se mostrarán:</p>

### Descarga de Snipe-it
<p style="text-align": justify;">Para descarga del hub de Docker la imagen más reciente de Snipe-it en Docker se usa el siguiente comando:
<br>`docker pull snipe/snipe-it`</br>
Y para comprobar que efectivamente se completó la descarga y esté disponible en Docker se usa el siguiente comando:
<br>`docker images`</br>
El cuál al ejecutarlo nos mostrará todas las imágenes disponibles, siendo la de nombre snipe/snipe-it la de mayor interés por este momento:
![docker_images](imagenes/docker_images.png)</p>

### Configuración de las variables de entorno
Snipe-it requiere de la creación de un archivo`.env`para poder funcionar. Para ello vamos a crear un archivo de extensión`.env`con el siguiente comando:

```
'nano nombre_del_archivo.env'
```

Se desplegará en la terminal el siguiente editor de texto:
![nano_vacio](imagenes/nano_vacio.png)

En el cuál pegaremos lo siguiente:

```
`# Mysql Parameters
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
PHP_UPLOAD_LIMIT=100`
```
#### Parámetros de MySQL
<p style="text-align": justify;">Para configurar los parámetros de MySQL vamos a entender primero cada uno de ellos:
<br>`MYSQL_ROOT_PASSWORD=`</br>
Este parámetro indica la contraseña que va a pedir MySQL cuando se quiera acceder al contenedor del mismo por medio de la consola de comandos para modificar las bases de datos, permisos, usuarios, etc.

<br>`MYSQL_DATABASE=`</br>
Es el nombre de la base de datos de MySQL que Snipe-it va a crear y usar. Es recomendable dejarla con el valor de `snipeit`.

<br>`MYSQL_USER=`</br> 
Es el usuario por default a crear en MySQL.

<br>`MYSQL_PASSWORD=`</br>
Es la contraseña del usuario creado anteriormente.</p>

#### Opciones de Snipe-it
En este apartado vamos a cambiar lo siguiente:
<br>`APP_URL=http://127.0.0.1:YOUR_PORT_NUMBER`</br>
<p style="text-align": justify;">En lugar de `127.0.0.1`colocaremos la dirección ip del servidor de pruebas Sofía, el cuál al día 17 de febrero de 2023 es `192.168.119.69`y cómo puerto se pondrá uno desponible (preguntar a los encargados de servicio social sobre qué puertos usar y cuáles no).</p>
<p style="text-align": justify;">`APP_TIMEZONE=US/Pacific`y`APP_LOCALE`especifican la zona horaria y el idioma de la aplicación respectivamente. Para la Ciudad de México la zona horaria es `America/Mexico_City` y el idioma es `es-ES`</p>

<p style="text-align": justify;">¡Y listo! Ya tenemos configuradas las variables de entorno de nuestra aplicación, por lo que podemos empezar a correr el contenedor MySQL, procedimiento que se verá a continuación.</p>

### Creación e inicio del contenedor con MYSQL 5.6
<p style="text-align": justify;"> El comando para iniciar el contenedor MySQL es el siguiente:
<br>`docker run --name snipe-mysql --env-file=my_env_file --mount source=snipesql-vol,target=/var/lib/mysql -d -P mysql:5.6`</br> 
Donde: 
*`--name` indica el nombre del contenedor.
*`--env-file`es el nombre del archivo.
*`.env` que creamos en un principio.
*`snipesql-vol` es el volumen de Snipe-it que se crea para MySQL.
*`-d` es la indicación para ocultar el log del comando que se ejecutó.
*`mysql:` es la versión de MySQL a usar.

>ATENCIÓN: Es importante usar la versión de MySQL menor o igual a 5.6, pues si se usa una mayor Snipe-it no va a conectarse con MySQL debido a que estas corren en modo estricto por default.
</p>

<p style="text-align": justify;">Una vez puesto en marcha el comando, la terminal debería de mostrar lo siguiente: </p>
![mysql_start](imagenes/mysql_start.png)
<p style="text-align": justify;">Los números que entrega la terminal es el ID del contendor. Para revisar si está en linea usamos el comando `docker ps` y debería de aparecer la ID que vimos anteriormente con el nombre del contenedor que pusimos en el comando de creación del contenedor MYSQL, en este caso: `snipe-mysql`.</p>
![docker_ps_1](imagenes/docker_ps_1.png)
<p style="text-align": justify;">Cómo se muestra en la terminal, quiere decir que nuestra base de datos ya se encuentra en línea en el servidor y lista para ser usada por Snipe-it.</p>
[Inicio](#índice)
## Inicio del contenedor Snipe-it
<p style="text-align": justify;">Iniciar el contenedor Snipe-it requiere de la generación de un APP_KEY, el cuál se va a generar mediante el siguiente comando:
<br>`docker run --rm snipe/snipe-it`</br>
No nos va a dejar iniciar el contenedor, pues nos dice que hace falta generar un APP_KEY para usar en el archivo .env, seguido de un ejemplo de APP_KEY que perfectamente podemos usar:</p>
![app_key](imagenes/app_key.png)
Vamos a copiar la clave que generó la terminal y la vamos a pegar en el apartado `APP_KEY=` que está dentro del archivo `.env` que creamos y editamos hace unos momentos:
![app_key_env](imagenes/app_key_env.png)

<p style="text-align": justify;">Una vez hecho esto, podemos correr nuestro contenedor Snipe-it con el siguiente comando:
</br>`docker run -d -p YOUR_PORT_NUMBER:80 --name="snipeit" --link snipe-mysql:mysql --env-file=my_env_file --mount source=snipe-vol,dst=/var/lib/snipeit snipe/snipe-it`</br>
donde:
*`YOUR_PORT_NUMBER` es el puerto que previamente colocamos en el archivo `.env`.
*`snipeit` es el nombre que le vamos a poner al contenedor
*`--link snipe-mysql:mysql` es la indicación de que vamos a conectar el contenedor de Snipe-it con el contenedor de MySQL que creamos anteriormente.
*`--env-file`es el archivo `.env` que creamos previamente.
</p>
De tal suerte que si lo editamos correctamente, vamos a poder ver lo siguiente en la terminal:
![snipeit_start](imagenes/snipeit_start.png)
Y al usar nuevamente el comando `docker ps`apreciaremos que está en línea el contenedor Snipe-it:
[docker_ps_2](imagenes/docker_ps_2.png)
[Inicio](#índice)

## Resultado final
<p style="text-align": justify;">Si hicimos todo bien, al ingresar a la dirección que pusimos en nuestro archivo `.env` desde nuestro navegador preferido, podremos ver el pre-flight de Snipe-it, es decir, la configuración inicial de la app en si de manera mucho más intuitiva.</p>
![snipeit_web](imagenes/snipeit_web.png)
<p style="text-align": justify;">Aquí nos indicará Snipe-it si hay algún problema con la configuración inicial, se mostrará de color rojo junto con la descripción del problema. Para consultar los problemas más comunes con su solución podemos visitar el FAQ de Snipe-it con la siguiente liga:
<a href="https://snipe-it.readme.io/docs/common-issues">Problema comunes con la instalación de Snipe-it</a>
</p>
<p style="text-align": justify;">Al continuar con los siguientes pasos en la página de Snipe-it, llegaremos a una página donde tenemos que configurar aspectos básicos de la aplicación, cómo lo son:
*Nombre del sitio de Snipe-it, por ejemplo: Adminisitración de recursos USECAD.
*Idioma de la app.
*Moneda.
*Extensión de los ID's de los recursos.</p>

<p style="text-align": justify;">Así como el dominio de los correos que se van a usar en la aplicación y el estilo de los correos a partir de los nombres de las personas. También pedirá nombre y apellido para el perfil del usuario inicial, su correo, usuario a usar dentro de Snipe-it y una contraseña para ingresara a dicho perfil.</p>
![snipeit_config](imagenes/snipeit_config.png)

Al terminar con la configuración, puede aparecer `500 web server error`, no pasa nada, solo basta con entrar de nuevo a la dirección ip y debería de aparecer la página de inicio de Snipe-it.
![snipeit_home](imagenes/snipeit_home.png)
¡Y listoo! Ya podremos usar Snipe-it junto con todas las maravillas de gestión que nos ofrece.

[Inicio](#índice)








