# AliUS
CREATE TABLE users (
    id INT(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE tutors (
    id INT(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT(11) UNSIGNED NOT NULL,
    bio TEXT,
    hourly_rate DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE subjects (
    id INT(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE sessions (
    id INT(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id INT(11) UNSIGNED NOT NULL,
    tutor_id INT(11) UNSIGNED NOT NULL,
    subject_id INT(11) UNSIGNED NOT NULL,
    date_time DATETIME NOT NULL,
    duration INT(11) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (student_id) REFERENCES users(id),
    FOREIGN KEY (tutor_id) REFERENCES tutors(id),
    FOREIGN KEY (subject_id) REFERENCES subjects(id)
);
<?php
session_start();

require_once 'config.php';

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $name = $_POST['name'];
    $email = $_POST['email'];
    $password = $_POST['password'];
    $user_type = $_POST['user_type'];

    // validate input fields
    if (empty($name) || empty($email) || empty($password) || empty($user_type)) {
        $_SESSION['error'] = 'All fields are required.';
        header('Location: register.php');
        exit;
    }

    // check if email already exists in the database
    $stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?');
    $stmt->execute([$email]);
    $user = $stmt->fetch();

    if ($user) {
        $_SESSION['error'] = 'Email already exists.';
        header('Location: register.php');
        exit;
    }

    // hash and salt password
    $salt = bin2hex(random_bytes(32));
    $hashed_password = hash('sha256', $password . $salt);

    // insert user into database
    $stmt = $pdo->prepare('INSERT INTO users (name, email, password, salt, user_type) VALUES (?, ?, ?, ?, ?)');
    $stmt->execute([$name, $email, $hashed_password, $salt, $user_type]);

    $_SESSION['success'] = 'Registration successful. Please login.';
    header('Location: login.php');
    exit;
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Register</title>
</head>
<body>
    <?php if (isset($_SESSION['error'])): ?>
        <div><?php echo $_SESSION['error']; ?></div>
        <?php unset($_SESSION['error']); ?>
    <?php endif; ?>

    <form method="POST">
        <label>Name:</label><br>
        <input type="text" name="name"><br>

        <label>Email:</label><br>
        <input type="email" name="email"><br>

        <label>Password:</label><br>
        <input type="password" name="password"><br>

        <label>User Type:</label><br>
        <select name="user_type">
            <option value="">Select User Type</option>
            <option value="user">User</option>
            <option value="tutor">Tutor</option>
        </select><br>

        <input type="submit" value="Register">
    </form>
</body>
</html>
<?php
session_start();

require_once 'config.php';

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $email = $_POST['email'];
    $password = $_POST['password'];

    // validate input fields
    if (empty($email) || empty($password)) {
        $_SESSION['error'] = 'All fields are required.';
        header('Location: login.php');
        exit;
    }

    // get user from database
    $stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?');
    $stmt->execute([$email]);
    $user = $stmt->fetch();

    // verify password
    if ($user && hash('sha256', $password . $user['salt']) === $user['password']) {
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['user_type'] = $user['user_type'];
        header('Location: dashboard.php');
        exit;
    } else {
        $_SESSION['error'] = 'Invalid email or password.';
        header('Location: login.php');
        exit;
    }
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <?php if (isset($_SESSION['error'])): ?>
        <div><?php echo $_SESSION['error']; ?></div>
        <?php unset($_SESSION['error']); ?>
    <?php endif; ?>

    <form method="POST">
        <label>Email:</label><br>
        <input type="email" name="email"><br>

        <label>Password:</label><br>
        <input type="password" name="password"><br>

        <input type="submit" value="Login">
    </form>
</body>
</html>
<form method="GET" action="search.php">
  <label for="subject">Subject:</label>
  <input type="text" name="subject" id="subject" required>

  <label for="availability">Availability:</label>
  <input type="text" name="availability" id="availability" required>

  <label for="location">Location:</label>
  <input type="text" name="location" id="location" required>

  <button type="submit">Search</button>
</form>
// Get search parameters from GET request
$subject = $_GET['subject'];
$availability = $_GET['availability'];
$location = $_GET['location'];

// Query database for tutors matching search parameters
$query = "SELECT * FROM tutors WHERE subject='$subject' AND availability='$availability' AND location='$location'";
$result = mysqli_query($conn, $query);

// Display search results
if (mysqli_num_rows($result) > 0) {
  while ($row = mysqli_fetch_assoc($result)) {
    echo "<h2>" . $row['name'] . "</h2>";
    echo "<p>Subject: " . $row['subject'] . "</p>";
    echo "<p>Availability: " . $row['availability'] . "</p>";
    echo "<p>Location: " . $row['location'] . "</p>";
    echo "<p>Rating: " . $row['rating'] . "</p>";
    echo "<a href='message.php?tutor_id=" . $row['id'] . "'>Contact Tutor</a>";
  }
} else {
  echo "No tutors found.";
}
// Add rating to tutor
$tutor_id = $_POST['tutor_id'];
$rating = $_POST['rating'];

$query = "UPDATE tutors SET rating=rating+$rating WHERE id=$tutor_id";
mysqli_query($conn, $query);

// Display tutor's new rating
$query = "SELECT rating FROM tutors WHERE id=$tutor_id";
$result = mysqli_query($conn, $query);
$row = mysqli_fetch_assoc($result);
echo "Tutor's rating is now " . $row['rating'];
// Display messages between user and tutor
$tutor_id = $_GET['tutor_id'];

$query = "SELECT * FROM messages WHERE tutor_id=$tutor_id";
$result = mysqli_query($conn, $query);

while ($row = mysqli_fetch_assoc($result)) {
  echo "<p>" . $row['message'] . "</p>";
}

// Send message to tutor
$message = $_POST['message'];

$query = "INSERT INTO messages (tutor_id, message) VALUES ($tutor_id, '$message')";
mysqli_query($conn, $query);

echo "Message sent.";]
