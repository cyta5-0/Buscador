# Buscador
Crear un buscador para artículos de cyta
🗂 Paso 1: Asegurar índices FULLTEXT
Primero, necesitás agregar índices FULLTEXT en los campos que vas a buscar. En este caso, podés crear uno combinado para:

sql
Copiar
Editar
ALTER TABLE dc
ADD FULLTEXT(title, description, creator_1, creator_2, creator_3,
             subject_1, subject_2, subject_3, subject_4, subject_5,
             subject_6, subject_7, subject_8, subject_9, subject_10,
             subject_11, subject_12);
⚠️ Nota: FULLTEXT funciona mejor en tablas MyISAM o InnoDB en MySQL 5.6+.

🧩 Paso 2: Script PHP del buscador (buscar_dc.php)
php
Copiar
Editar
<?php
// Conexión a la base de datos
$mysqli = new mysqli("localhost", "usuario", "contraseña", "basededatos");

if ($mysqli->connect_error) {
    die("Error de conexión: " . $mysqli->connect_error);
}

// Capturar término de búsqueda
$busqueda = $_GET['q'] ?? '';

$resultados = [];

if (!empty($busqueda)) {
    $stmt = $mysqli->prepare("
        SELECT identifier, title, description, date 
        FROM dc 
        WHERE MATCH(title, description, creator_1, creator_2, creator_3,
                    subject_1, subject_2, subject_3, subject_4, subject_5,
                    subject_6, subject_7, subject_8, subject_9, subject_10,
                    subject_11, subject_12) 
              AGAINST (? IN NATURAL LANGUAGE MODE)
        LIMIT 50
    ");
    $stmt->bind_param("s", $busqueda);
    $stmt->execute();
    $resultados = $stmt->get_result();
}
?>

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Buscador CyTA</title>
</head>
<body>
    <h1>Buscador de artículos en CyTA</h1>
    <form method="get" action="buscar_dc.php">
        <input type="text" name="q" value="<?= htmlspecialchars($busqueda) ?>" placeholder="Buscar...">
        <button type="submit">Buscar</button>
    </form>

    <?php if ($busqueda): ?>
        <h2>Resultados para: <em><?= htmlspecialchars($busqueda) ?></em></h2>
        <?php if ($resultados->num_rows > 0): ?>
            <ul>
                <?php while ($row = $resultados->fetch_assoc()): ?>
                    <li>
                        <strong><a href="https://www.cyta.com.ar/ta/article.php?id=<?= $row['identifier'] ?>"><?= htmlspecialchars($row['title']) ?></a></strong><br>
                        <?= mb_strimwidth(strip_tags($row['description']), 0, 200, "...") ?><br>
                        <small><?= $row['date'] ?></small>
                    </li>
                <?php endwhile; ?>
            </ul>
        <?php else: ?>
            <p>No se encontraron resultados.</p>
        <?php endif; ?>
    <?php endif; ?>
</body>
</html>
🚩 Paso 3: Publicarlo en CyTA
Podés subir el archivo como buscar_dc.php y enlazarlo desde la home o desde las páginas de artículos. Una buena ubicación podría ser en el menú lateral o en la parte superior como "🔍 Buscar artículos".

💡 Bonus
Luego podés ir agregando:

Filtros por date, type, creator

Resultados ordenados por relevancia o fecha

Búsqueda avanzada (AND, OR, frases entre comillas)
