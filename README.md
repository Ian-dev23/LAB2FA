Implementación de Autenticación de Dos Factores (2FA) con PHP, MySQL y Google Authenticator
Universidad Tecnológica de Panamá
Desarrollo de Software VII
1. INTRODUCCIÓN

La autenticación de dos factores (2FA) es un mecanismo de seguridad que agrega una segunda capa de protección al proceso de inicio de sesión.

Además de ingresar usuario y contraseña, el usuario debe proporcionar un código temporal generado por la aplicación Google Authenticator.

Esto reduce significativamente el riesgo de accesos no autorizados incluso si la contraseña es comprometida.

2. OBJETIVO

Implementar un sistema de autenticación de dos factores utilizando:

PHP
MySQL
Google Authenticator
Librería de autenticación TOTP
3. TECNOLOGÍAS UTILIZADAS
Tecnología	Descripción
PHP	Lenguaje de programación del servidor
MySQL	Base de datos
WAMP	Entorno de desarrollo local
Composer	Gestor de dependencias PHP
Google Authenticator	Aplicación generadora de códigos TOTP
Google2FA	Librería para validar códigos
4. CONFIGURACIÓN DE LA BASE DE DATOS

Se creó la base de datos:

CREATE DATABASE ejemplologin;
Tabla Usuarios
CREATE TABLE usuarios(
    id INT AUTO_INCREMENT PRIMARY KEY,
    Usuario VARCHAR(100),
    Password VARCHAR(255),
    HashMagic VARCHAR(255),
    secret_2fa VARCHAR(255)
);

Cumpliendo con el requerimiento:

ALTER TABLE usuarios
ADD secret_2fa VARCHAR(255) NULL AFTER HashMagic;
Tabla Intentos de Login
CREATE TABLE intentos_login(
    id INT AUTO_INCREMENT PRIMARY KEY,
    Usuario VARCHAR(100),
    ipRemoto VARCHAR(50),
    deteccion_anomalia INT
);
5. USUARIO DE BASE DE DATOS CON PRIVILEGIOS MÍNIMOS

Se creó un usuario específico para la aplicación.

CREATE USER 'login_user'@'localhost'
IDENTIFIED BY 'Login2026!';

Se otorgaron únicamente los privilegios necesarios:

GRANT SELECT, INSERT, UPDATE
ON ejemplologin.*
TO 'login_user'@'localhost';

Verificación de privilegios:

SHOW GRANTS FOR 'login_user'@'localhost';
6. INSTALACIÓN DE DEPENDENCIAS

Se utilizó Composer para instalar la librería.

composer require pragmarx/google2fa

Posteriormente se generó el directorio:

vendor/

y el archivo:

vendor/autoload.php
7. FORMULARIO DE REGISTRO

Se desarrolló un formulario para registrar colaboradores.

Campos implementados:

Nombre
Apellido
Correo
Sexo
Contraseña
Validaciones implementadas
Frontend
Campos obligatorios.
Correo válido.
Contraseña requerida.
Backend
Sanitización de datos.
Validación de duplicados.
Generación segura de hash.
8. SANITIZACIÓN DE DATOS

Se implementó la clase:

SanitizarEntrada

Métodos utilizados:

limpiarCadena()
limpiarCorreo()
limpiarNumero()

Estos métodos eliminan caracteres potencialmente peligrosos y reducen el riesgo de ataques.

9. GENERACIÓN DE HASH

Las contraseñas nunca son almacenadas en texto plano.

Se utiliza:

password_hash()

Ejemplo:

$hash = password_hash(
    $password,
    PASSWORD_BCRYPT
);

Validación:

password_verify(
    $password,
    $hash
);
10. GENERACIÓN DEL SECRET 2FA

Durante el registro del colaborador se genera automáticamente un secreto único.

$google2fa = new Google2FA();

$secret = $google2fa->generateSecretKey();

Este valor es almacenado en:

secret_2fa
11. GENERACIÓN DEL CÓDIGO QR

Se genera una URL compatible con Google Authenticator.

$qr = $google2fa->getQRCodeUrl(
    'EjemploLogin',
    $correo,
    $secret
);

Posteriormente se genera el código QR dinámicamente.

<img src="https://api.qrserver.com/v1/create-qr-code/?size=250x250&data=<?php echo $qr; ?>">
12. REGISTRO DEL COLABORADOR

Proceso:

Usuario llena formulario.
Sistema valida datos.
Sistema genera hash.
Sistema genera secret_2fa.
Sistema almacena información.
Sistema muestra QR.
Usuario escanea QR con Google Authenticator.
13. INICIO DE SESIÓN

El usuario ingresa:

Usuario
Contraseña

El sistema:

Busca el usuario.
Obtiene el HashMagic.
Valida la contraseña.
password_verify()

Si es correcta:

$_SESSION["Usuario"]

es creada.

14. VALIDACIÓN DEL SEGUNDO FACTOR

Luego del login exitoso:

redireccionar("validar2fa.php");

El usuario introduce el código generado por Google Authenticator.

Ejemplo:

$valid = $google2fa->verifyKey(
    $_SESSION["secret_2fa"],
    $codigo
);
Código Correcto

Se crea la sesión:

$_SESSION["autenticado"] = "SI";

y se permite el acceso al sistema.

Código Incorrecto

Se muestra:

Código incorrecto

y el acceso es denegado.

15. CONTROL DE SESIONES

Sesiones utilizadas:

$_SESSION["Usuario"]
$_SESSION["secret_2fa"]
$_SESSION["autenticado"]

Estas sesiones garantizan que únicamente usuarios autenticados puedan ingresar al panel.

16. REGISTRO DE INTENTOS DE LOGIN

Cada intento es almacenado en la tabla:

intentos_login

Datos registrados:

Usuario
Dirección IP
Resultado del intento

Ejemplo:

registrarIntentos();

Esto permite detectar comportamientos anómalos.

17. FLUJO COMPLETO DEL SISTEMA
Registro
Formulario Registro
        ↓
Validación Datos
        ↓
Hash Password
        ↓
Generar Secret 2FA
        ↓
Guardar Usuario
        ↓
Generar QR
        ↓
Escanear QR
Inicio de Sesión
Login
        ↓
Validar Usuario
        ↓
Validar Password
        ↓
Pantalla 2FA
        ↓
Ingresar Código
        ↓
Validar TOTP
        ↓
Panel de Control
18. PRUEBAS REALIZADAS
Prueba 1

Registro exitoso de colaborador.

Resultado:

✅ Correcto

Prueba 2

Generación automática de secret_2fa.

Resultado:

✅ Correcto

Prueba 3

Generación de QR.

Resultado:

✅ Correcto

Prueba 4

Escaneo con Google Authenticator.

Resultado:

✅ Correcto

Prueba 5

Inicio de sesión con contraseña válida.

Resultado:

✅ Correcto

Prueba 6

Ingreso de código TOTP válido.

Resultado:

✅ Correcto

Prueba 7

Ingreso de código TOTP inválido.

Resultado:

✅ Acceso denegado

19. CONCLUSIONES
Se implementó correctamente la autenticación de dos factores.
Cada usuario posee un secreto único para Google Authenticator.
Las contraseñas son almacenadas mediante hash seguro.
Se registran los intentos de autenticación.
El sistema requiere la validación del código TOTP antes de permitir el acceso.
Se incrementó significativamente el nivel de seguridad del sistema.
20. COMANDOS IMPORTANTES UTILIZADOS
composer require pragmarx/google2fa
ALTER TABLE usuarios
ADD secret_2fa VARCHAR(255) NULL AFTER HashMagic;
SHOW GRANTS FOR 'login_user'@'localhost';
password_hash()
password_verify()
generateSecretKey()
verifyKey()
session_start()
session_destroy()
