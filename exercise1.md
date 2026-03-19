# Exercise 1 - File Handling in PHP

In this exercise, you will learn how to handle files in PHP using functions like `fopen()`, `fread()`, `fgets()`, `fwrite()`, and `fclose()`. Each exercise builds on the previous one to help you understand file handling step by step.

---

## Exercise 1.1: Checking if a File Exists
1. Write a PHP script to check if a file named `example.txt` exists.
2. Use the `file_exists()` function to perform the check.
3. Display a message indicating whether the file exists or not.

### Explanation
The `file_exists()` function checks if a file exists at the specified path. It returns `true` if the file exists and `false` otherwise.

### Sample Code
```php
<?php
$filename = "example.txt";
if (file_exists($filename)) {
    echo "The file $filename exists.";
} else {
    echo "The file $filename does not exist.";
}
?>
```

---

## Exercise 1.2: Opening and Reading a File
1. Create a PHP script to open and read the contents of `example.txt`.
2. Use the `fopen()` function to open the file in read mode (`r`).
3. Use the `fread()` function to read the entire file.
4. Close the file using `fclose()`.

### Explanation
- `fopen()` opens a file and returns a **file handle** — a resource that PHP uses internally to track the open file's position and state. You pass this handle to other functions like `fread()` and `fclose()`.
- `fread($file, $length)` reads up to `$length` bytes from the file. Passing `filesize($filename)` as the length reads the entire file at once.
- `fclose()` closes the file and releases the system resources associated with it. Always close files after you are done with them.
- `nl2br()` converts newline characters (`\n`) in a string to HTML `<br>` tags so that line breaks are visible in the browser.

### File Modes Reference
When using `fopen()`, the second argument specifies how the file should be opened:

| Mode | Description |
|------|-------------|
| `r`  | Read only. File pointer starts at the beginning. |
| `w`  | Write only. Truncates (empties) the file, or creates it if it doesn't exist. |
| `a`  | Append only. File pointer starts at the end, or creates the file if it doesn't exist. |
| `x`  | Write only. Creates the file; fails if the file already exists. |
| `r+` | Read and write. File pointer starts at the beginning. |
| `w+` | Read and write. Truncates the file, or creates it if it doesn't exist. |
| `a+` | Read and append. File pointer starts at the end, or creates the file if it doesn't exist. |

### Sample Code
```php
<?php
$filename = "example.txt";
if (file_exists($filename)) {
    $file = fopen($filename, "r"); // Open the file in read mode
    $contents = fread($file, filesize($filename)); // Read the entire file
    fclose($file); // Close the file
    echo nl2br($contents); // Display the contents
} else {
    echo "The file $filename does not exist.";
}
?>
```

---

## Exercise 1.3: Reading a File Line by Line
1. Modify the script from Exercise 1.2 to read the file line by line.
2. Use the `fgets()` function to read each line.
3. Display each line on the webpage.

### Explanation
- `fgets()` reads one line from the file at a time, up to (and including) the newline character.
- This is useful for large files where reading the entire file at once would consume too much memory.
- The condition `!== false` uses **strict comparison** (type and value). `fgets()` returns `false` when it reaches the end of the file (EOF) or encounters a read error. Because `fgets()` can also return an empty string `""` for an empty line, using `!=` instead of `!==` would incorrectly treat an empty line as `false` and stop the loop early. Using `!== false` ensures the loop continues until `fgets()` genuinely returns `false` (EOF or error).

### Sample Code
```php
<?php
$filename = "example.txt";
if (file_exists($filename)) {
    $file = fopen($filename, "r");
    while (($line = fgets($file)) !== false) { // Read each line
        echo nl2br($line); // Display the line
    }
    fclose($file);
} else {
    echo "The file $filename does not exist.";
}
?>
```

---

## Exercise 1.4: Writing to a File
1. Create a PHP script to write user input to a file named `output.txt`.
2. Use the `fopen()` function to open the file in write mode (`w`).
3. Use the `fwrite()` function to write data to the file.
4. Close the file using `fclose()`.

### Explanation
- Opening a file in write mode (`w`) **overwrites** (truncates) any existing content in the file. If the file does not exist, it will be created automatically.
- Use `fwrite()` to write data to the file.
- `PHP_EOL` is a built-in PHP constant that represents the correct end-of-line character(s) for the operating system the script is running on (`\r\n` on Windows, `\n` on Linux/macOS). Using `PHP_EOL` instead of a hard-coded `"\n"` makes your code portable across platforms.

### Sample Code
```php
<!-- HTML Form -->
<form method="post">
    <input type="text" name="user_input" placeholder="Enter text">
    <button type="submit">Submit</button>
</form>

<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $input = $_POST["user_input"];
    $file = fopen("output.txt", "w"); // Open the file in write mode
    fwrite($file, $input . PHP_EOL); // Write the input to the file
    fclose($file); // Close the file
    echo "Data written to file.";
}
?>
```

---

## Exercise 1.5: Appending to a File
1. Modify the script from Exercise 1.4 to append user input to the file instead of overwriting it.
2. Use the `fopen()` function to open the file in append mode (`a`).

### Explanation
- Opening a file in append mode (`a`) adds new data to the **end** of the file without erasing existing content. This is the key difference from write mode (`w`), which truncates the file first.
- If the file does not yet exist, append mode will create it, just like write mode.
- All other aspects (using `fwrite()`, `fclose()`, etc.) remain the same as in Exercise 1.4.

### Sample Code
```php
<!-- HTML Form -->
<form method="post">
    <input type="text" name="user_input" placeholder="Enter text">
    <button type="submit">Submit</button>
</form>

<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $input = $_POST["user_input"];
    $file = fopen("output.txt", "a"); // Open the file in append mode
    fwrite($file, $input . PHP_EOL); // Append the input to the file
    fclose($file); // Close the file
    echo "Data appended to file.";
}
?>
```

---

## Exercise 1.6: Deleting a File
1. Write a PHP script to delete a file named `temp.txt`.
2. Use the `unlink()` function to delete the file.
3. Ensure the script checks if the file exists before attempting to delete it.

### Explanation
The `unlink()` function deletes a file. Always check if the file exists before calling `unlink()` to avoid errors.

### Sample Code
```php
<?php
$filename = "temp.txt";
if (file_exists($filename)) {
    unlink($filename); // Delete the file
    echo "The file $filename has been deleted.";
} else {
    echo "The file $filename does not exist.";
}
?>
```