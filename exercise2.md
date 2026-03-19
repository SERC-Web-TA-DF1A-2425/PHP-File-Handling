# Exercise 2 - File Uploads

In this exercise, you will learn how to handle file uploads in PHP. Each exercise builds on the previous one to help you understand file upload handling step by step.

---

## Understanding the `$_FILES` Superglobal
The `$_FILES` superglobal is an associative array in PHP that stores information about files uploaded via an HTML form. It contains the following keys for each uploaded file:
- `name`: The original name of the uploaded file.
- `type`: The MIME type of the uploaded file (e.g., `image/jpeg`).
- `tmp_name`: The temporary file path where the uploaded file is stored on the server.
- `error`: An error code indicating the status of the upload (e.g., `0` for no error).
- `size`: The size of the uploaded file in bytes.

### Example
```php
<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    echo "File Name: " . $_FILES["uploaded_file"]["name"] . "<br>";
    echo "File Type: " . $_FILES["uploaded_file"]["type"] . "<br>";
    echo "Temporary Path: " . $_FILES["uploaded_file"]["tmp_name"] . "<br>";
    echo "Error Code: " . $_FILES["uploaded_file"]["error"] . "<br>";
    echo "File Size: " . $_FILES["uploaded_file"]["size"] . " bytes<br>";
}
?>
```

---

## Exercise 2.1: Basic File Upload
1. Create an HTML form to allow users to upload a file.
2. Write a PHP script to handle the file upload.
3. Save the uploaded file to a directory named `uploads/`.
4. Display a success message with the file name after the upload is complete.

### Explanation
- Use the `$_FILES` superglobal to access uploaded file information.
- Use the `move_uploaded_file()` function to move the uploaded file from its temporary server location to the desired directory. It also verifies that the file was legitimately uploaded via HTTP POST, which is an important security check.
- The HTML form requires `enctype="multipart/form-data"` to enable file uploads. Without this attribute, the browser sends only the file name, not the file contents.
- `basename()` extracts just the file name from a full path (e.g., `"uploads/photo.jpg"` → `"photo.jpg"`), preventing path traversal attacks where a malicious file name like `"../../config.php"` could overwrite sensitive files.
- `htmlspecialchars()` converts special characters (such as `<`, `>`, and `&`) to safe HTML entities before displaying them in the browser, protecting against Cross-Site Scripting (XSS) attacks.

### Sample Code
```php
<!-- HTML Form -->
<form method="post" enctype="multipart/form-data">
    <input type="file" name="uploaded_file">
    <button type="submit">Upload</button>
</form>

<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $targetDir = "uploads/";
    $targetFile = $targetDir . basename($_FILES["uploaded_file"]["name"]);

    if (move_uploaded_file($_FILES["uploaded_file"]["tmp_name"], $targetFile)) {
        echo "The file " . htmlspecialchars(basename($_FILES["uploaded_file"]["name"])) . " has been uploaded.";
    } else {
        echo "Sorry, there was an error uploading your file.";
    }
}
?>
```

---

## Exercise 2.2: Checking if the File Already Exists
1. Modify the script from Exercise 2.1 to check if the file already exists in the `uploads/` directory.
2. If the file exists, display a message and do not overwrite it.

### Explanation
- Use the `file_exists()` function to check if the file already exists.

### Sample Code
```php
// ...existing code...
$targetFile = $targetDir . basename($_FILES["uploaded_file"]["name"]);

if (file_exists($targetFile)) {
    echo "Sorry, the file already exists.";
} else {
    if (move_uploaded_file($_FILES["uploaded_file"]["tmp_name"], $targetFile)) {
        echo "The file " . htmlspecialchars(basename($_FILES["uploaded_file"]["name"])) . " has been uploaded.";
    } else {
        echo "Sorry, there was an error uploading your file.";
    }
}
```

---

## Exercise 2.3: Limiting File Size
1. Modify the script to limit the size of the uploaded file to 2MB.
2. If the file exceeds the size limit, display an error message.

### Explanation
- Use the `$_FILES["uploaded_file"]["size"]` property to check the file size.
- 2MB = 2 * 1024 * 1024 bytes. There are 1024 bytes in a kilobyte, and 1024 kilobytes in a megabyte.
- This check uses the file size reported by the browser. For a more robust check, you can also configure the `upload_max_filesize` and `post_max_size` directives in `php.ini` to enforce server-side limits regardless of what the script does.

### Sample Code
```php
// ...existing code...
$maxFileSize = 2 * 1024 * 1024; // 2MB

if ($_FILES["uploaded_file"]["size"] > $maxFileSize) {
    echo "Sorry, your file is too large. Maximum allowed size is 2MB.";
} else {
    // ...existing code for file upload...
}
```

### Modifying PHP Configuration for Larger File Uploads

PHP enforces its own server-side upload limits independently of any checks in your script. If you need to allow files larger than the defaults, you must update two directives in your `php.ini` file:

| Directive | Default | Description |
|-----------|---------|-------------|
| `upload_max_filesize` | `2M` | Maximum size of a single uploaded file. |
| `post_max_size` | `8M` | Maximum size of the entire POST request body (must be larger than `upload_max_filesize`). |

**Steps to update `php.ini`:**

1. Find the location of the active `php.ini` file by running the following command in your terminal:
   ```bash
   php --ini
   ```
   Look for the line starting with `Loaded Configuration File`.

2. Open the file in a text editor and locate the two directives. Update them to the values you need, for example to allow up to 20MB uploads:
   ```ini
   upload_max_filesize = 20M
   post_max_size = 24M
   ```
   > **Note:** `post_max_size` should always be larger than `upload_max_filesize` to leave room for the other form fields in the POST body.

3. Save the file and restart your web server for the changes to take effect:
   - **Apache:** `sudo systemctl restart apache2`
   - **Nginx + PHP-FPM:** `sudo systemctl restart php-fpm`
   - **Built-in PHP development server:** simply stop and restart the process.

4. Verify the new limits are active by creating a small PHP file with:
   ```php
   <?php
   echo 'upload_max_filesize: ' . ini_get('upload_max_filesize') . '<br>';
   echo 'post_max_size: ' . ini_get('post_max_size');
   ```

**Alternative: using `.htaccess` for shared hosting**

If you do not have access to `php.ini` (e.g., on a shared host), you can use a `.htaccess` file in your project root to override the limits. Note that `upload_max_filesize` and `post_max_size` are `PHP_INI_PERDIR` directives, meaning they can only be changed in `php.ini`, `.htaccess`, or `httpd.conf` — **not** at runtime with `ini_set()`. For shared hosting, use a `.htaccess` file instead:
```apacheconf
php_value upload_max_filesize 20M
php_value post_max_size 24M
```

---

## Exercise 2.4: Restricting File Types
1. Modify the script to allow only specific file types (e.g., `.jpg`, `.png`, `.gif`).
2. If the file type is not allowed, display an error message.

### Explanation
- `pathinfo($path, PATHINFO_EXTENSION)` extracts the file extension from a file path. For example, `pathinfo("photo.JPG", PATHINFO_EXTENSION)` returns `"JPG"`.
- `strtolower()` converts the extension to lowercase so that comparisons work regardless of how the user capitalised the extension (e.g., `.JPG`, `.Jpg`, and `.jpg` all become `"jpg"`).
- `in_array($value, $array)` checks whether `$value` exists in `$array`. Here it checks whether the uploaded file's extension is in the list of allowed types.
- **Note:** Checking the file extension alone is not fully secure because a malicious user could rename a file to have an allowed extension. For stronger validation, also check the file's MIME type using `$_FILES["uploaded_file"]["type"]` or the `mime_content_type()` function.

### Sample Code
```php
// ...existing code...
$allowedTypes = ["jpg", "png", "gif"];
$fileType = strtolower(pathinfo($targetFile, PATHINFO_EXTENSION));

if (!in_array($fileType, $allowedTypes)) {
    echo "Sorry, only JPG, PNG, and GIF files are allowed.";
} else {
    // ...existing code for file upload...
}
```

---

## Exercise 2.5: Combining All Checks
1. Combine the checks from Exercises 2.2, 2.3, and 2.4 into a single script.
2. Ensure the script checks for file existence, size, and type before uploading the file.

### Explanation
- Perform all checks sequentially before calling `move_uploaded_file()`.
- The sample code checks existence first, then size, then file type. Rejecting invalid file types and oversized files early avoids unnecessary filesystem operations, so consider performing those checks before the existence check in larger applications.
- Only call `move_uploaded_file()` after all validation has passed to ensure that only safe, correctly-sized, correctly-typed files are saved to the server.

### Sample Code
```php
<!-- HTML Form -->
<form method="post" enctype="multipart/form-data">
    <input type="file" name="uploaded_file">
    <button type="submit">Upload</button>
</form>

<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $targetDir = "uploads/";
    $targetFile = $targetDir . basename($_FILES["uploaded_file"]["name"]);
    $maxFileSize = 2 * 1024 * 1024; // 2MB
    $allowedTypes = ["jpg", "png", "gif"];
    $fileType = strtolower(pathinfo($targetFile, PATHINFO_EXTENSION));

    // Check if file already exists
    if (file_exists($targetFile)) {
        echo "Sorry, the file already exists.";
    }
    // Check file size
    elseif ($_FILES["uploaded_file"]["size"] > $maxFileSize) {
        echo "Sorry, your file is too large. Maximum allowed size is 2MB.";
    }
    // Check file type
    elseif (!in_array($fileType, $allowedTypes)) {
        echo "Sorry, only JPG, PNG, and GIF files are allowed.";
    }
    // Upload the file
    else {
        if (move_uploaded_file($_FILES["uploaded_file"]["tmp_name"], $targetFile)) {
            echo "The file " . htmlspecialchars(basename($_FILES["uploaded_file"]["name"])) . " has been uploaded.";
        } else {
            echo "Sorry, there was an error uploading your file.";
        }
    }
}
?>
