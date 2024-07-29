# last-value


The zip file contains the following files:

1. get_last_movement.php : Likely a PHP script to fetch the latest movement data.
2. icon.png:An image file, probably an icon used in the web interface.
3. index.html :The main HTML file, likely the homepage or interface of the project.
4. record_movement.php : Another PHP script, likely used to record movement data.

Here's a detailed explanation of each file:

### 1. get_last_movement.php

This PHP script retrieves the most recent movement direction from a database.

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

$db_host = "localhost";
$db_user = "root";
$db_pass = ""; 
$db_name = "my_database"; 

try {
    $conn = new mysqli($db_host, $db_user, $db_pass, $db_name);

    if ($conn->connect_error) {
        throw new Exception("Connection failed: " . $conn->connect_error);
    }

    $sql = "SELECT direction FROM movement_table ORDER BY timestamp DESC LIMIT 1";
    $result = $conn->query($sql);

    if ($result->num_rows > 0) {
        $row = $result->fetch_assoc();
        echo $row["direction"];
    } else {
        echo "None";
    }

    $conn->close();

} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
}
?>
```

- **Purpose**: Connects to the database, fetches the latest movement direction, and displays it.
- **Database**: Assumes a MySQL database with a `movement_table` that records movements.
- **Error Handling**: Displays errors if there are issues connecting to the database or executing the query.

### 2. icon.png

This is an image file, typically used as an icon for the web interface. It does not contain any code or text to explain.

### 3. index.html

This HTML file is the main interface for controlling movement and displaying the last recorded movement.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Movement Control</title>
    <link rel="icon" type="image/png" href="icon.png">
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
        }
        .button-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(3, 1fr);
            grid-gap: 10px;
            max-width: 300px;
            margin: 20px auto;
        }
        .button {
            background-color: #007bff;
            color: white;
            padding: 10px 20px;
            border: none;
            cursor: pointer;
            font-size: 16px;
        }
        .button:hover {
            background-color: #0056b3;
        }
        #forward { grid-area: 1 / 2 / 2 / 3; }
        #left { grid-area: 2 / 1 / 3 / 2; }
        #stop { grid-area: 2 / 2 / 3 / 3; background-color: #dc3545; }
        #right { grid-area: 2 / 3 / 3 / 4; }
        #backward { grid-area: 3 / 2 / 4 / 3; }
        #stop:hover { background-color: #a71d2a; }
        #lastMovement {
            margin-top: 20px;
            font-size: 18px;
        }
    </style>
</head>
<body>
    <h1>Movement Control Interface</h1>
    <div class="button-container">
        <button id="forward" class="button" onclick="recordMovement('Forward')">Forward</button>
        <button id="left" class="button" onclick="recordMovement('Left')">Left</button>
        <button id="stop" class="button" onclick="recordMovement('Stop')">Stop</button>
        <button id="right" class="button" onclick="recordMovement('Right')">Right</button>
        <button id="backward" class="button" onclick="recordMovement('Backward')">Backward</button>
    </div>
    <div id="lastMovement">Loading...</div>
    <script>
        function recordMovement(direction) {
            var xhr = new XMLHttpRequest();
            xhr.open("POST", "record_movement.php", true);
            xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
            xhr.onload = function() {
                if (xhr.status === 200) {
                    updateLastMovement(direction);
                } else {
                    console.error("Error: " + xhr.status + " " + xhr.statusText);
                }
            };
            xhr.onerror = function() {
                console.error("Connection error");
            };
            xhr.send("direction=" + encodeURIComponent(direction));
        }

        function getLastMovement() {
            var xhr = new XMLHttpRequest();
            xhr.open("GET", "get_last_movement.php", true);
            xhr.onload = function() {
                if (xhr.status === 200) {
                    updateLastMovement(xhr.responseText);
                } else {
                    console.error("Error: " + xhr.status + " " + xhr.statusText);
                }
            };
            xhr.onerror = function() {
                console.error("Connection error");
            };
            xhr.send();
        }

        function updateLastMovement(direction) {
            var lastMovementElement = document.getElementById('lastMovement');
            lastMovementElement.textContent = "Last Movement: " + direction;
        }

        window.onload = getLastMovement;
    </script>
</body>
</html>
```

- **Purpose**: Provides a web interface to control movement directions and display the last movement.
- **Layout**: Uses a grid layout to arrange buttons for movement directions (Forward, Left, Stop, Right, Backward).
- **JavaScript**: 
  - `recordMovement(direction)`: Sends the selected movement direction to `record_movement.php` via POST request.
  - `getLastMovement()`: Fetches the last movement direction from `get_last_movement.php` via GET request.
  - `updateLastMovement(direction)`: Updates the displayed last movement direction.

### 4. `record_movement.php`

This PHP script records a movement direction into the database.

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

$db_host = "localhost";
$db_user = "root";
$db_pass = ""; 
$db_name = "my_database"; 

try {
    $conn = new mysqli($db_host, $db_user, $db_pass, $db_name);

    if ($conn->connect_error) {
        throw new Exception("Connection failed: " . $conn->connect_error);
    }

    if (!isset($_POST["direction"])) {
        throw new Exception("No direction data received");
    }

    $direction = $_POST["direction"];

    $stmt = $conn->prepare("INSERT INTO movement_table (direction) VALUES (?)");
    if (!$stmt) {
        throw new Exception("Error preparing statement: " . $conn->error);
    }

    $stmt->bind_param("s", $direction);

    if (!$stmt->execute()) {
        throw new Exception("Error executing statement: " . $stmt->error);
    }

    echo "Movement recorded successfully";

    $stmt->close();
    $conn->close();

} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
}
?>
```

-  **Purpose** : Connects to the database and inserts a new movement direction.
- **Database**: Inserts a record into `movement_table`.
- **Error Handling** : Provides error messages for connection issues, missing data, statement preparation errors, and execution errors.

These files together create a simple web interface for recording and displaying movement directions using a database backend.


# Result 

https://github.com/user-attachments/assets/9b2a4871-396d-4f89-bd03-47609eccb3e2

