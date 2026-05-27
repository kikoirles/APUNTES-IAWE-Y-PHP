# Programación avanzada en PHP

## Introducción

Más allá de los aspectos fundamentales para trabajar con el lenguaje PHP, existen funciones o procedimientos que actualmente son necesarios para la creación de aplicaciones web. Entre estos aspectos se incluye, el cifrado de la información, la gestión de sesiones o el almacenamiento de información en el cliente, entre otros.

## Cifrado de la infomación

PHP dispone de una serie de funciones que permiten cifrar la información para aumentar la seguridad de las aplicaciones web que se desarrollen. Estas funciones manejan una serie de algoritmos de cifrado que pueden ser modificados en diferentes versiones de PHP dependiendo de las vulnerabilidades encontradas en estos algoritmos o la apararición de nuevos algoritmos que aumenten la dificultad para descifrarlos.

### Almacenamiento de información cifrada en la B.D.

Para almacenar información cifrada en la base de datos se puede utilizar la función *password_hash* que se implementa en PHP de forma nátiva y que incorpora retrocompatibilidad con funciones implementadas en versiones anteriores de PHP. El prototipo de la función es el siguiente:

```php
password_hash ( string $textoacifrar, integer $algoritmo , array $opciones = ? ) : string
```

* La función recibe la contraseña en el primer parámetro en texto plano mediante una cadena de caracteres.
* El segundo parámetro permite definir el algoritmo de cifrado utilizado. Podemos definirlo mediante las siguientes constantes:
  * **PASSWORD_DEFAULT**. Usa el algoritmo bcrypt, establecido por defecto desde PHP5. Utilizando este parámetro si las nuevas versiones de PHP implementan nuevos algoritmos, se utilizarán estos de forma predeterminada.
  * **PASSWORD_BCRYPT**. Usa el algoritmo *CRYPT_BLOWFISH* para crear el hash y este es retrocompatible con versiones anteriores de PHP.
* El tercer parámetro permite establecer unas opciones de cifrado que se explican a continuación, aunque no sea recomendado su uso en la actualidad:
  * **salt**. Se proporciona manualmente una sal a usar cuando se realiza el hash de la contraseña. La sal es una cadena alfanumerica aleatoria que dificulta el proceso de descifrar la información por fuerza bruta. Es recomendable dejar que PHP defina la sal de forma aleatoria.
  * **cost**. Denota el coste del algoritmo que debería usarse, es un entero que define el número de pasadas que realizará el algoritmo y que por defecto tiene un valor de 10

A continuación, se muestra un ejemplo del uso de esta función para almacenar contraseñas en la base de datos.

```php
<?php
  $con=mysqli_connect('mariadb', 'miguel', 'leugim', 'iawe');
	if($con!=FALSE){
    $pass= password_hash('leugim',PASSWORD_DEFAULT); //Generamos la contraseña cifrada
    $sentencia = mysqli_prepare($con, "INSERT INTO usuario(id_usurio, nombre, pass) VALUES(1,'miguel',?)");
    mysqli_stmt_bind_param($sentencia, "s", $pass);
    $resultado = mysqli_stmt_execute($sentencia);
  }else{
    echo "¡ERROR! No se ha podido conectar a la base de datos."
?>
```

### Recuperación de la información cifrada de la base de datos

La segundad de las operaciones que tenemos que tener en cuenta es el descifrado de la información que tenemos almacenada en la base de datos. Para comparar los datos que tenemos almacenados con aquellos que recibamos por parámetros se utiliza la función *password_verify*. A continución, se muestra el prototipo de esta función para desencriptar los datos almacenados.

```php
password_verify ( string $textoadescifrar , string $hashadescifrar ) : bool
```

Los parámetros que recibe esta función son:

* La cadena de caracteres que hemos recibido y con la que queremos realizar la comparación de lo que tenemos almacenado en la base de datos.
* El hash que tenemos almacenado en la base de datos y con el cual realizaremos la comparación.

En el siguiente ejemplo se puede ver la utilización de esta función para comprobar, en un proceso de login, que la contraseña es correcta.

```php
<?php
  $con=mysqli_connect('mariadb', 'miguel', 'leugim', 'iawe');
	if($con!=FALSE){
    $sentencia = mysqli_prepare($con, "SELECT pass FROM usuario WHERE id_usuario = ?");
    mysqli_stmt_bind_param($sentencia, "i", 1);
    $pass_hash= mysqli_stmt_execute($sentencia);
    if(password_verify ('leugim',$pass_hash)){
      echo "Login realizado correctamente";
    }else{
      echo "¡ERROR! Contraseña incorrecta";
    }
  }else{
    echo "¡ERROR! No se ha podido conectar a la base de datos."
?>
```

## La gestión de cookies con PHP

Una cookie permite almacenar información en el cliente mediante un par clave->valor. Utilizando este procedimiento existe la posibilidad de almacenar información para poder recuperarla posteriormente o para poder traspasarla de una página PHP a otra. 

Hay que tener cuidad cuando se utiliza este método para almacenar información, pues, al hacerlo en el cliente no existe control alguno, ni seguridad de que estos datos puedan almacenarse. También hay que considerar que el cliente puede no permitir la grabación de sesiones en su navegador y que pueden ser borradas y modificadas por lo que los datos podrían ser incorrectos.

Debemos tener en consideración que las cookies `solo pueden almacenar valores en formato cadena de caracteres`. También hay que considerar que el proceso de crear una cookie debe ser instanciado antes de que el script php genere ningún tipo de salida. sto requiere que se realice la llamada a esta función antes de cualquier salida, incluyendo etiquetas `<html>` y `<head>` así como cualquier espacio en blanco.

### Grabación de información mediante cookies

Para almacenar información mediante cookies utilizamos la función *setcookie* de PHP. Podemos utilizar la función *setcookie* utilizando la función de cuyo prototipo aparece a continuación:

```php
setcookie ( string $clave , string $valor = "" , int $fechaexpiracion = 0 , string $ruta = "" , string $dominio = "" , bool $seguro = false , bool $solohttp = false ) : bool
```

Los parámetros que recibe la función son los siguientes:

* **$clave**. Es el nombre con el que podremos obtener el valor de la cookie.
* **$valor**. El el valor asociado al nombre que será almacenado en la cookie.
* **$fechaexpiracion**. Por defecto la cookie se borra cuando se cierra el navegador. Con este parámetro podemos aumentar el tiempo de almacenamiento en el cliente definiendo la fecha donde caducará. Podemos utilizar la función *time()* para definir la fecha actual más un incremento, como por ejemplo time()+600.
* **$ruta**. Define una ruta dentro del directorio del cliente donde se almacenan.
* **$dominio**. Define un dominio o subdominio donde la cookie también estará disponible. Por defecto se utiliza el dominio al que tengamos asociado la página web.
* **$seguro**. Con valor *true* esta cookie solo se almacenará si se está utilizando una conexión segura. El valor por defecto será *false*.
* **$solohttp**. Con el valor a *true* solo se permitirá acceder a las cookies desde el protocolo *http* y no de otros accesos como *javascript*. Por defecto el valor de este parámetro es *false*.

A continuación se muestra un ejemplo de utilización de esta función dentro de una página web.

```php
<?php
$visitas = '1';
//Almacenamos en el navegador el número de visitas.
setcookie("n_visitas", $visitas); 
//Crea la cookie que se borrará una hora después de haber sido almacenada
setcookie("n_visitas", $visitas, time()+3600); 
//Crea la cookie que se borrará en una hora dentro del directorio miguel que servirá para todo el dominio de amigos del panda y que solo se almacenará si estamos accediendo mediante una conexión segura.
setcookie("n_visitas", $visitas, time()+3600, "/miguel/", "amigosdelpanda.com", 1);
?>
```

### Aceeso al valor almacenado por una cookie

Para realizar el proceso contrario y leer las cookies almacenadas se utiliza el array *$_COOKIE* que es un array asociativo al que podremos acceder por la clave de la cookie al valor que hemos almacenado en el navegador cliente. En el siguiente ejemplos se puede ver el método para acceder a la cookie definida en el apartado anterior.

```php
<?php
// Imprime una cookie individual
echo $_COOKIE["n_visitas"];

// Podemos mostrar el valor de todas las cookies aplicando la función print_r a todo el array $_COOKIE
print_r($_COOKIE);
?>
```

### Array de cookies 

En ocasiones puede utilizarse la notación de arrays para almacenar las cookies en el cliente, aunque igualmente tedioso que almacenarlas de la forma vista anteriormente, este método proporcionará ventajas a la hora de realizar la lectura de las mismas.

En el ejemplo siguiente se muestra como utilizar este  proceso.

```php
<?php
// Se almacena la información del usuario en notación de array
setcookie("datosUsuario[nombre]", "miguel");
setcookie("datosUsuario[pass]", "leugim");
setcookie("datosUsuario[nick]", "mike");

// Se recorre la cookie como si fuera un array asociativo obteniendo la clave y el valor y se muestran
if (isset($_COOKIE['datosUsuario'])) {
    foreach ($_COOKIE['datosUsuario'] as $clave => $valor) {
        $clave = htmlspecialchars($clave);
        $valor = htmlspecialchars($valor);
        echo "$clave : $valor <br />\n";
    }
}
?>
```

### Consideraciones del uso de cookies

La utilización de cookies debe ser uno de los últimos recursos a los que se debe recurrir. Este procedimiento es muy utilizado dentro de la web para realizar un rastreo de las páginas web donde el cliente ha accedido y el endurecimiento de las políticas de privacidad sobre todo en la UE en los últimos años obligan a informar al usuario de la información que se está almacenando. 

Adicionalmente, se están proponiendo nuevas alternativas que no utilicen las cookies dentro de las páginas web como la FLOC propuesta por Google.

* [Información oficial de Google FLOC](https://blog.google/products/ads-commerce/2021-01-privacy-sandbox/)
* [La guerra de las cookies - Xataka](https://www.xataka.com/privacidad/google-quiere-acabar-cookies-internet-dice-que-su-alternativa-casi-igual-eficaz-para-anunciantes)

## La gestión de sesiones

Las sesiones permiten la compartición de la información entre dos páginas PHP sin tener que almacenar esta información en el navegador cliente. Al igual que cuando utilizamos cookies, accedemos a la información que se ha almacenado mediante un array asociativo denominado *$_SESSION* que almacenará pares clave, valor. 

A diferencia de las cookies, las sesiones se borran cuando se ejecuta en el servidor un cierre de sesión o estas caducan con el tiempo y ofrecen mayor seguridad que las cookies para compartir información entre varias páginas PHP. Esta forma de compartir la información entre dos páginas PHP no permite acceder a datos de forma común, siendo el principal uso de las sesiones para establecer que un usuario ha iniciado sesión.

Para definir una sesión, esta debe crearse de forma explícita y destruirla o cerrarla cuando ya no sea necesario su uso. Las sesiones se crean y utilizan con la función *session_start()* que nos devolverá un identificador aleatorio de la sesión, por contra, la función *session_destroy()* eliminará la sesión creada. 

Por norma general, estas funciones se utilizan sin un parámetro para definir el nombre de la sesión y todas las páginas utilizan el mismo identificador de sesión creado de forma aleatoria. A continuación, se muestran ejemplos de los procedimientos para iniciar, utilizar y destruir una sesión.

### Creación de una sesión

La creación de una sesión es una instrucción que al igual que las cookies viaja en la cabecera HTML y que por lo tanto se debe crear al inicio del script php, antes de cualquier instrucción de salida o etiqueta HTML.

```php
<?php
session_start();
//Almacenamos la información dentro del array identificándolo mediante una etiqueta
$_SESSION["nombre_usuario"] = "miguel";
echo "Se ha iniciado sesión con el usuario ".$_SESSION["nombre_usuario"];
?>
```

En ningún caso podremos establecer la instrucción *session_start()* después de realizar un echo, print o imprimir cualquier código HTML.

### Uso de una sesión

Para utilizar una sesión tendremos que utilizar la instrucción *session_start()* para poder acceder al array $_SESSION y recuperar el valor que hemos almacenado en el paso anterior. En el ejemplo siguiente se muestra la forma de acceder a los varoles que se han almacenado en el paso anterior.

```php
<?php
session_start();
//Recuperamos el valor almacenado dentro del array con la clave nombre_usuario.
echo "Estas logeado como: ".$_SESSION["nombre_usuario"];
?>
```

El array asociativo $SESSION funciona como un array normal y podemos almacenar arrays asociados a claves, tal y como se muestra en el ejemplo siguiente:

```php
<?php
session_start();
$_SESSION["datos_usuario"][] = array("Miguel Ángel","Tomás Amat", "miguel","84.58.73.144");

//Recuperamos el valor almacenado dentro del array con la clave nombre_usuario y podemos acceder mediante el índice
echo "Estas logeado como: ".$_SESSION["nombre_usuario"][2];
echo "Tu nombre es: ".$_SESSION["nombre_usuario"][0];
echo "Tus apellidos son: ".$_SESSION["nombre_usuario"][0];
echo "Estás conectado desde la IP: ".$_SESSION["nombre_usuario"][3];
?>
```

### Destrucción de la sesión

Cuando se destruye una sesión dentro de una página PHP, esta puede seguir accediendo al array $_SESSION pero el resto de páginas no podrán acceder a este array. Este proceso puede ocurrir cuando se llama a la función *session_destroy()* de forma explicita, cuando el usuario cierra la sesión y termina la conexión con el servidor o cuando se cumple el tiempo de expiración de la sesión que se indica en la directiva *session.gc_maxlifetime*. A continuación se muestra un ejemplo de la destrucción de la sesión de forma explícita dentro de una script PHP.

```php
<?php
session_start();
$_SESSION["id_usuario"]=475;
$_SESSION["nombre"]="miguel";
$_SESSION["logeado"]=true;

//Con la siguiente instrucción otros scripts php no podran acceder a las variables $_SESSION["logeado"] y el resto definidas al inicio del ejemplo. Como el destroy también debe establecerse antes de cualquier otra instrucción php, estos valores si podrán ser accedidos desde este mismo fichero php
session_destroy();

if ($_SESSION["logeado"]){
  echo "El usuario está logeado ".$_SESSION["nombre"]" está logeado."
}

//Obtener los datos del usuario
$con=mysqli_connect('mariadb', 'miguel', 'leugim', 'iawe');
if($con!=FALSE){
    $sentenciaPreparada = mysqli_prepare($con, "SELECT * FROM usuario WHERE id_usuario = ?");
    mysqli_stmt_bind_param($sentenciaPreparada, "i", $_SESSION["id_usuario"]=475);
    $resultado = mysqli_stmt_execute($sentenciaPreparada);
}


?>
```

### Borrar elementos del array $_SESSION

Otra de las acciones que podemos considerar al trabajar con sesiones es eliminar valores que ya se hayan establecido. Para ello utilizamos la función *unset()* al que le pasaremos el identificador de la variable. Tal y como se muestra el ejemplo siguiente desde el momento que se ejecuta ya no se podrá acceder a los valores identificados por esta clave dentro del array.

```php
<?php
session_start();
//Almacenamos la información dentro del array identificándolo mediante una etiqueta
$_SESSION["nombre_usuario"] = "miguel";
echo "Se ha iniciado sesión con el usuario ".$_SESSION["nombre_usuario"];

unset($_SESSION["nombre_usuario"]);

//Esta instrucción provocaría un error al destruir la variable $_SESSION["nombre_usuario"]
echo "Se ha iniciado sesión con el usuario ".$_SESSION["nombre_usuario"];
?>
```
