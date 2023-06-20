# EXPLOIT LAB SETUP

First , we we setup our attacker domain at: https://subdomain.target.com

Now create a `.htaccess` file to route the requests on the server:

The `.htaccess` file code is:

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ log.php [L]
```
This `.htaccess` code redirects all requests for non-existent files and directories to `log.php` setting up a front controller pattern.

We create the `log.php` referenced by the `.htaccess`

The code for the `log.php` is:

```
<?php
$logFile = 'requests.txt';

$method = $_SERVER['REQUEST_METHOD'];
$uri = $_SERVER['REQUEST_URI'];
$queryString = $_SERVER['QUERY_STRING'];
$loggedParams = $uri . '?' . $queryString;
$ip = $_SERVER['REMOTE_ADDR'];

// Log the request to the file
file_put_contents($logFile, "$method $loggedParams [IP: $ip]\n", FILE_APPEND);

// Alert the logged parameters
echo "<script>alert('ATTACKER SUCCESSFULLY LOGGED YOUR IP AND: " . htmlspecialchars($loggedParams, ENT_QUOTES) . "');</script>";
?>
```

Now this code logs the HTTP request method, requested URI with query parameters, and the client’s IP address to a file, and displays an alert in the browser indicating that an attacker has successfully logged the IP and the requested parameters.

We now create ATTACKER DASHBOARD to log all requests and steal tokens as well as handle XSS rendering.

The code for a simple dashboard is:

```
<!DOCTYPE html>
<html>
<head>
<title>Admin Panel</title>
<style>
body {
  background-color: #000;
  color: #0f0;
  font-family: 'Courier New', monospace;
  padding: 20px;
}

pre {
  background-color: #000;
  padding: 10px;
  overflow: auto;
  white-space: pre-wrap;
  border-radius: 5px;
  max-height: 400px;
  border: 1px solid #0f0;
}

.header {
  font-size: 24px;
  margin-bottom: 20px;
}

.button {
  background-color: #0f0;
  color: #000;
  border: none;
  padding: 8px 16px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin-right: 10px;
  cursor: pointer;
}
</style>
</head>
<body>
<h1 class="header">Request Logs</h1>
<pre id="console"></pre>
<div>
<button class="button" onclick="refreshLogs()">Refresh</button>
<button class="button" onclick="clearLogs()">Clear Logs</button>
<a href="#" class="button" onclick="exportLogs()">Export</a>
</div>
<script>
// Function to fetch and display the log file contents
function fetchLogs() {
  fetch('requests.txt')
    .then(response => response.text())
    .then(data => {
      const consoleElement = document.getElementById('console');
      consoleElement.textContent = escapeHtml(data);
      consoleElement.scrollTop = consoleElement.scrollHeight;
    })
    .catch(error => console.log(error));
}

// Escape HTML entities
function escapeHtml(text) {
  const element = document.createElement('div');
  element.textContent = text;
  return element.innerHTML;
}

// Refresh logs button click handler
function refreshLogs() {
  fetchLogs();
}

// Clear logs button click handler
function clearLogs() {
  fetch('clear_logs.php', { method: 'POST' })
    .then(response => {
      if (response.ok) {
        console.log('Logs cleared successfully');
        fetchLogs();
      }
    })
    .catch(error => console.log(error));
}

// Export logs button click handler
function exportLogs() {
  const link = document.createElement('a');
  link.href = 'requests.txt';
  link.download = 'requests.txt';
  link.click();
}

// Initial fetch of the log file contents
fetchLogs();
</script>
</body>
</html>
```

That just shows you a panel to load, refresh and delete stolen credential records.

Now I created a `clear_logs.php` file referenced above for the Clear Logs button. Its code is:

```
<?php
$logFile = 'requests.txt';

// Clear the log file by truncating it
file_put_contents($logFile, '');

// Return a response indicating success
http_response_code(200);
echo 'Logs cleared successfully';
?>
```

## NOW WE EXPLOIT!!
