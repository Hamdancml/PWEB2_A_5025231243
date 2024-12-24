# PWEB2_A_5025231243
# Nama : Ahmad Izzul Hamdan Zoelfa
# NRP : 5025231243
# Kelas : PWEB-A

1. Index.php
   
               <?php
          session_start();
          
          // Check if the user is logged in (either by session or cookie)
          if (!isset($_SESSION['user_logged_in']) || $_SESSION['user_logged_in'] != true) {
              // Check if the username cookie is set
              if (isset($_COOKIE['username'])) {
                  $_SESSION['user_logged_in'] = true;
                  $_SESSION['username'] = $_COOKIE['username'];
              } else {
                  header('Location: login.php'); // Redirect to login if not logged in
                  exit();
              }
          }
          if (isset($_SESSION['user_logged_in']) && $_SESSION['user_logged_in'] == true) {
              echo "<p>Welcome, " . $_SESSION['username'] . "!</p>";
          } else {
              echo "<p>Welcome, Guest!</p>";
          }
          
          setcookie("user_visit", "1", time() + 3600, "/");
          
          include 'db_connect.php';
          ?>
          
          
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <title>Football Transfer System</title>
              <link rel="stylesheet" href="style.css">
          </head>
          <body>
          
          <div class="container">
              <h1>Football Transfer System</h1>
              <a href="addplayer.php" class="btn">Add New Player</a>
              <a href="upload.php" class="btn">Upload File</a>

              <h2>Transfer List</h2>
              <table>
                  <tr>
                      <th>ID</th>
                      <th>Player Name</th>
                      <th>From Club</th>
                      <th>To Club</th>
                      <th>Transfer Fee</th>
                      <th>Actions</th>
                  </tr>

                  <?php
                  $sql = "SELECT * FROM transfers";
                  $result = $conn->query($sql);
          
                  if ($result->num_rows > 0) {
                      while($row = $result->fetch_assoc()) {
                          echo "<tr>";
                          echo "<td>{$row['id']}</td>";
                          echo "<td>{$row['player_name']}</td>";
                          echo "<td>{$row['from_club']}</td>";
                          echo "<td>{$row['to_club']}</td>";
                          echo "<td>$" . number_format($row['transfer_fee'], 2) . "</td>";
                          echo "<td><a href='editplayer.php?id={$row['id']}'>Edit</a> | <a href='delete.php?id={$row['id']}'>Delete</a></td>";
                          echo "</tr>";
                      }
                  } else {
                      echo "<tr><td colspan='6'>No players found</td></tr>";
                  }
                  $conn->close();
                  ?>
              </table>
          
              <a href="logout.php" class="btn">Logout</a> <!-- Add Logout Button -->
          </div>
          
          <?php
          session_start();
          
          if (!isset($_SESSION['user_logged_in']) || $_SESSION['user_logged_in'] != true) {
              header("Location: login.php");
              exit();
          }
          ?>
          
          
          </body>
          </html>

2. Addplayer.php

                <?php
          include 'db_connect.php';
          
          if ($_SERVER['REQUEST_METHOD'] == 'POST') {
              $player_name = mysqli_real_escape_string($conn, $_POST['player_name']);
              $from_club = mysqli_real_escape_string($conn, $_POST['from_club']);
              $to_club = mysqli_real_escape_string($conn, $_POST['to_club']);
              $transfer_fee = mysqli_real_escape_string($conn, $_POST['transfer_fee']);

              $sql = "INSERT INTO transfers (player_name, from_club, to_club, transfer_fee) VALUES ('$player_name', '$from_club', '$to_club', '$transfer_fee')";
          
              if ($conn->query($sql) === TRUE) {
                  // Redirect to index.php after successful insertion
                  header("Location: index.php");
                  exit();
              } else {
                  echo "Error: " . $sql . "<br>" . $conn->error;
              }
          
              $conn->close();
          }
          ?>
          
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <title>Add New Player</title>
              <link rel="stylesheet" href="style.css">
          </head>
          <body>
          
          <div class="container">
              <h2>Add New Player</h2>
              <form method="post" action="addplayer.php">
                  <label for="player_name">Player Name:</label>
                  <input type="text" name="player_name" required>

                  <label for="from_club">From Club:</label>
                  <input type="text" name="from_club" required>
          
                  <label for="to_club">To Club:</label>
                  <input type="text" name="to_club" required>
          
                  <label for="transfer_fee">Transfer Fee:</label>
                  <input type="number" name="transfer_fee" required>
          
                  <input type="submit" value="Add Player">
              </form>
          </div>
          
          </body>
          </html>

3. db_connection.php

             <?php
          $servername = "localhost";
          $username = "root";
          $password = "";
          $dbname = "football_transfers";
          
          $conn = new mysqli($servername, $username, $password, $dbname);
          
          if ($conn->connect_error) {
              die("Connection failed: " . $conn->connect_error);
          }
          ?>

4. delete.php

             <?php
          session_start();
          include 'db_connect.php';
          
          // Check if the ID is set in the URL
          if (isset($_GET['id'])) {
              $id = $_GET['id'];

              // Delete the player from the database
              $delete_sql = "DELETE FROM transfers WHERE id = $id";
              
              if ($conn->query($delete_sql) === TRUE) {
                  echo "Record deleted successfully!";
                  header('Location: index.php'); // Redirect back to the list
              } else {
                  echo "Error deleting record: " . $conn->error;
              }
          } else {
              echo "No ID specified.";
              exit();
          }
          
          $conn->close();
          ?>

5. editplayer.php

             <?php
          session_start();
          include 'db_connect.php';
          
          // Check if the ID is set in the URL
          if (isset($_GET['id'])) {
              $id = $_GET['id'];
          
              // Fetch the player's details from the database
              $sql = "SELECT * FROM transfers WHERE id = $id";
              $result = $conn->query($sql);
              if ($result->num_rows == 1) {
                  $row = $result->fetch_assoc();
              } else {
                  echo "Player not found.";
                  exit();
              }
          
              // Update the player's information
              if ($_SERVER['REQUEST_METHOD'] == 'POST') {
                  $player_name = $_POST['player_name'];
                  $from_club = $_POST['from_club'];
                  $to_club = $_POST['to_club'];
                  $transfer_fee = $_POST['transfer_fee'];
          
                  $update_sql = "UPDATE transfers SET player_name='$player_name', from_club='$from_club', to_club='$to_club', transfer_fee='$transfer_fee' WHERE id = $id";
                  
                  if ($conn->query($update_sql) === TRUE) {
                      echo "Record updated successfully!";
                      header('Location: index.php'); // Redirect back to the list
                  } else {
                      echo "Error updating record: " . $conn->error;
                  }
              }
          } else {
              echo "No ID specified.";
              exit();
          }
          
          $conn->close();
          ?>
          
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <title>Edit Player</title>
          </head>
          <body>
          
          <h1>Edit Player</h1>
          <form method="POST">
              <label for="player_name">Player Name:</label>
              <input type="text" name="player_name" value="<?php echo $row['player_name']; ?>" required><br>
          
              <label for="from_club">From Club:</label>
              <input type="text" name="from_club" value="<?php echo $row['from_club']; ?>" required><br>
          
              <label for="to_club">To Club:</label>
              <input type="text" name="to_club" value="<?php echo $row['to_club']; ?>" required><br>
          
              <label for="transfer_fee">Transfer Fee:</label>
              <input type="number" name="transfer_fee" value="<?php echo $row['transfer_fee']; ?>" required><br>
          
              <button type="submit">Update Player</button>
          </form>
          
          </body>
          </html>

6. login.php

             <?php
          session_start();
          include 'db_connect.php'; // Pastikan ini adalah koneksi database Anda
          
          if ($_SERVER['REQUEST_METHOD'] == 'POST') {
              $username = $_POST['username'];
              $password = $_POST['password'];
          
              // Validasi input
              if (!empty($username) && !empty($password)) {
                  // Query untuk mencari pengguna
                  $sql = "SELECT * FROM users WHERE username = ?";
                  $stmt = $conn->prepare($sql);
                  $stmt->bind_param("s", $username);
                  $stmt->execute();
                  $result = $stmt->get_result();
          
                  if ($result->num_rows > 0) {
                      $user = $result->fetch_assoc();
          
                      // Verifikasi password
                      if (password_verify($password, $user['password'])) {
                          // Login berhasil, set session
                          $_SESSION['user_logged_in'] = true;
                          $_SESSION['username'] = $user['username'];
          
                          // Set cookie jika diinginkan
                          if (isset($_POST['remember_me'])) {
                              setcookie("username", $user['username'], time() + (86400 * 7), "/");
                          }
          
                          // Redirect ke index.php
                          header("Location: index.php");
                          exit();
                      } else {
                          $error = "Password salah!";
                      }
                  } else {
                      $error = "Username tidak ditemukan!";
                  }
              } else {
                  $error = "Username dan password tidak boleh kosong!";
              }
          }
          ?>
          
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <title>Login</title>
              <style>
                  body {
                      font-family: Arial, sans-serif;
                      background-color: #333;
                      color: #fff;
                      display: flex;
                      justify-content: center;
                      align-items: center;
                      height: 100vh;
                      margin: 0;
                  }
          
                  .login-container {
                      background-color: rgba(0, 0, 0, 0.8);
                      padding: 20px;
                      border-radius: 8px;
                      width: 300px;
                      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
                  }
          
                  input[type="text"], input[type="password"] {
                      width: 100%;
                      padding: 10px;
                      margin: 10px 0;
                      border: none;
                      border-radius: 4px;
                  }
          
                  button {
                      width: 100%;
                      padding: 10px;
                      background-color: #007bff;
                      border: none;
                      border-radius: 4px;
                      color: #fff;
                      font-weight: bold;
                  }
          
                  button:hover {
                      background-color: #0056b3;
                  }
          
                  .error {
                      color: red;
                      margin-top: 10px;
                  }
              </style>
          </head>
          <body>
          
          <div class="login-container">
              <h2>Login</h2>
              <form action="login.php" method="POST">
                  <input type="text" name="username" placeholder="Username" required>
                  <input type="password" name="password" placeholder="Password" required>
                  <label>
                      <input type="checkbox" name="remember_me"> Remember Me
                  </label>
                  <button type="submit">Login</button>
              </form>
              <?php if (isset($error)): ?>
                  <p class="error"><?php echo $error; ?></p>
              <?php endif; ?>
          </div>
          
          </body>
          </html>

7. logout.php

          <?php
          session_start();
          session_unset();
          session_destroy();
          
          // Hapus cookie jika ada
          if (isset($_COOKIE['username'])) {
              setcookie("username", "", time() - 3600, "/");
          }
          
          // Redirect ke login
          header("Location: login.php");
          exit();
          ?>

8. register.php

          <?php
          session_start();
          include 'db_connect.php';
          
          // Handle registration submission
          if ($_SERVER['REQUEST_METHOD'] == 'POST') {
              $username = $_POST['username'];
              $password = $_POST['password'];
              $confirm_password = $_POST['confirm_password'];
          
              // Check if the passwords match
              if ($password != $confirm_password) {
                  $error_message = "Passwords do not match!";
              } else {
                  // Hash the password before saving it
                  $hashed_password = password_hash($password, PASSWORD_DEFAULT);
          
                  // Only check for username existence after passwords match
                  $sql = "SELECT * FROM users WHERE username = ?";
                  $stmt = $conn->prepare($sql);
                  $stmt->bind_param("s", $username); // Bind the username to the query
                  $stmt->execute();
                  $result = $stmt->get_result();
          
                  if ($result->num_rows > 0) {
                      $error_message = "Username already exists!";
                  } else {
                      // Insert the new user into the database if no duplicate username
                      $insert_sql = "INSERT INTO users (username, password) VALUES (?, ?)";
                      $insert_stmt = $conn->prepare($insert_sql);
                      $insert_stmt->bind_param("ss", $username, $hashed_password); // Bind the parameters
                      if ($insert_stmt->execute()) {
                          // Redirect to login.php after successful registration
                          header('Location: login.php');
                          exit();
                      } else {
                          $error_message = "Error registering user: " . $conn->error;
                      }
                  }
              }
          }
          
          $conn->close();
          ?>
          
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <title>Register</title>
          </head>
          <body>
          
          <h1>Create a New Account</h1>
          
          <?php
          if (isset($error_message)) {
              echo "<p style='color: red;'>$error_message</p>";
          }
          ?>
          
          <form method="POST">
              <label for="username">Username:</label>
              <input type="text" name="username" required><br>
          
              <label for="password">Password:</label>
              <input type="password" name="password" required><br>
          
              <label for="confirm_password">Confirm Password:</label>
              <input type="password" name="confirm_password" required><br>
          
              <button type="submit">Register</button>
          </form>
          
          <p>Already have an account? <a href="login.php">Login here</a></p>
          
          </body>
          </html>
          <?php
          session_start();
          include 'db_connect.php';
          
          // Handle registration submission
          if ($_SERVER['REQUEST_METHOD'] == 'POST') {
              $username = $_POST['username'];
              $password = $_POST['password'];
              $confirm_password = $_POST['confirm_password'];
          
              // Check if the passwords match
              if ($password != $confirm_password) {
                  $error_message = "Passwords do not match!";
              } else {
                  // Hash the password before saving it
                  $hashed_password = password_hash($password, PASSWORD_DEFAULT);
          
                  // Only check for username existence after passwords match
                  $sql = "SELECT * FROM users WHERE username = ?";
                  $stmt = $conn->prepare($sql);
                  $stmt->bind_param("s", $username); // Bind the username to the query
                  $stmt->execute();
                  $result = $stmt->get_result();
          
                  if ($result->num_rows > 0) {
                      $error_message = "Username already exists!";
                  } else {
                      // Insert the new user into the database if no duplicate username
                      $insert_sql = "INSERT INTO users (username, password) VALUES (?, ?)";
                      $insert_stmt = $conn->prepare($insert_sql);
                      $insert_stmt->bind_param("ss", $username, $hashed_password); // Bind the parameters
                      if ($insert_stmt->execute()) {
                          // Redirect to login.php after successful registration
                          header('Location: login.php');
                          exit();
                      } else {
                          $error_message = "Error registering user: " . $conn->error;
                      }
                  }
              }
          }
          
          $conn->close();
          ?>
          
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <title>Register</title>
          </head>
          <body>
          
          <h1>Create a New Account</h1>
          
          <?php
          if (isset($error_message)) {
              echo "<p style='color: red;'>$error_message</p>";
          }
          ?>
          
          <form method="POST">
              <label for="username">Username:</label>
              <input type="text" name="username" required><br>
          
              <label for="password">Password:</label>
              <input type="password" name="password" required><br>
          
              <label for="confirm_password">Confirm Password:</label>
              <input type="password" name="confirm_password" required><br>
          
              <button type="submit">Register</button>
          </form>
          
          <p>Already have an account? <a href="login.php">Login here</a></p>
          
          </body>
          </html>

9. style.css
          
             body {
              font-family: Arial, sans-serif;
              background-color: #f4f4f9;
              display: flex;
              justify-content: center;
              align-items: center;
              min-height: 100vh;
              padding: 20px;
          }
          
          .container {
              max-width: 800px;
              background: white;
              padding: 20px;
              border-radius: 8px;
              box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
          }
          
          h1, h2 {
              color: #333;
              margin-bottom: 20px;
              text-align: center;
          }
          
          table {
              width: 100%;
              border-collapse: collapse;
              margin-top: 20px;
          }
          
          table, th, td {
              border: 1px solid #ddd;
              padding: 10px;
          }
          
          th {
              background-color: #5d9cec;
              color: white;
          }
          
          .btn {
              display: inline-block;
              background-color: #5d9cec;
              color: white;
              padding: 10px 20px;
              text-decoration: none;
              border-radius: 4px;
              margin-right: 10px;
              text-align: center;
          }

10. upload.php

          <?php
          if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_FILES['file'])) {
              $upload_dir = 'uploads/';
              $file_path = $upload_dir . basename($_FILES['file']['name']);
          
              if (move_uploaded_file($_FILES['file']['tmp_name'], $file_path)) {
                  echo "File uploaded successfully";
              } else {
                  echo "Failed to upload file";
              }
          }
          ?>
          
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <title>Upload File</title>
              <link rel="stylesheet" href="style.css">
          </head>
          <body>
          
          <div class="container">
              <h2>Upload File</h2>
              <form action="upload.php" method="post" enctype="multipart/form-data">
                  <input type="file" name="file" required>
                  <input type="submit" value="Upload File">
              </form>
          </div>
          
          </body>
          </html>







   



