## Enlaces a los demás archivos

<img src ="https://i.pinimg.com/736x/74/0d/a7/740da76c140323c7be4cdd0577ca01b3.jpg" width ="50%">

[desvkill](devskill.md)

[generales](generales.md)

[interes](interes.md)

[softkill](softkill.md)

<?php
// procesar.php

// 1. Conexión a la base de datos MySQL usando la extensión MySQLi
$servername = "localhost";
$username = "root"; // En un entorno real de producción, NO usar root.
$password = "tu_contraseña_de_mysql"; // <-- ¡CAMBIA ESTO POR TU CONTRASEÑA REAL!
$dbname = "congreso_gomez_ortiz";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("La conexión ha fallado: " . $conn->connect_error);
}

// 2. Recibir y limpiar los datos del formulario (Validación Server-Side básica)
$nombre = trim($_POST['nombre']);
$apellido_paterno = trim($_POST['apellido_paterno']);
$apellido_materno = trim($_POST['apellido_materno']);
$correo = filter_var($_POST['correo'], FILTER_SANITIZE_EMAIL);
$matricula = trim($_POST['matricula']);
$telefono = trim($_POST['telefono']);
$rfc_input = trim($_POST['rfc']);

// Normalizar el RFC a mayúsculas como decisión de diseño
$rfc = strtoupper($rfc_input);

// Validación de correo server-side requerida por el reto
if (!filter_var($correo, FILTER_VALIDATE_EMAIL)) {
    die("Error: El formato del correo electrónico no es válido.");
}

// 3. Lógica del RFC (El problema del siglo)
$yearStr = substr($rfc, 4, 2);
$month = substr($rfc, 6, 2);
$day = substr($rfc, 8, 2);

$yearInt = (int)$yearStr;
$currentYear2Digits = (int)date('y'); // Año actual a dos dígitos (ej. 26 para 2026)

// Decisión sobre el siglo:
if ($yearInt > $currentYear2Digits) {
    $fullYear = 1900 + $yearInt; // Asumir siglo XX (19XX)
} else {
    $fullYear = 2000 + $yearInt; // Asumir siglo XXI (20XX)
}

$fecha_nacimiento = "$fullYear-$month-$day";

// 4. Calcular la edad exacta
$dob = new DateTime($fecha_nacimiento);
$now = new DateTime();
$edad = $now->diff($dob)->y;

// 5. Preparar la consulta SQL (Prevención de SQL Injection)
$stmt = $conn->prepare("INSERT INTO asistentes (nombre, apellido_paterno, apellido_materno, correo, matricula, telefono, rfc, fecha_nacimiento, edad) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)");

// La "s" significa String y la "i" significa Integer. Son 8 strings y 1 integer.
$stmt->bind_param("ssssssssi", $nombre, $apellido_paterno, $apellido_materno, $correo, $matricula, $telefono, $rfc, $fecha_nacimiento, $edad);

if ($stmt->execute()) {
    echo "<h2>Registro exitoso</h2>";
    echo "<p>Asistente registrado correctamente.</p>";
    echo "<p>Edad calculada: $edad años.</p>";
    echo "<a href='formulario.php'>Volver al formulario</a> | <a href='reporte.php'>Ver Reporte</a>";
} else {
    echo "Error al registrar: " . $stmt->error;
}

$stmt->close();
$conn->close();
?>
