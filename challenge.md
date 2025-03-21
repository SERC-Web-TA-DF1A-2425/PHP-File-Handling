# Challenge Exercise - Photo Gallery

In this exercise, you will create a simple photo gallery using PHP and HTML. The gallery will display images from a directory on the server. You will also add functionality to upload new images to the gallery.

---

## 1. Create HTML template

We will use a template for the common HTML structure for each page of the gallery.

1. Create a file named `template.php` with the following content:

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?php echo $page_title ?? "Photo Gallery"; ?></title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>Photo Gallery></h1>
            <nav>
                <a href="index.php">Home</a>
                <a href="gallery.php">Gallery</a>
                <a href="upload.php">Upload</a>
            </nav>
        </header>
        <main>
            <?php echo $content ?? ""; ?>
        </main>
        <footer>
            <p>&copy; <?php echo date("Y"); ?> Photo Gallery</p>
        </footer>
    </div>
</body>
</html>
```

2. Create a file named `styles.css` with the following content:

```css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f0f0f0;
}

.container {
    max-width: 960px;
    margin: 0 auto;
    padding: 1rem;
}
```

Add more styles as needed to customize the appearance of the gallery.

3. Create a file named `index.php` with the following content:

```php
<?php
$page_title = "Home";
ob_start();
?>

<h2>Welcome to the Photo Gallery</h2>
<p>This is a simple photo gallery created using PHP and HTML.</p>

<?php
$content = ob_get_clean();
include "template.php";
?>
```

Explanation:
- The `template.php` file contains the common structure for all pages of the gallery.
- The `styles.css` file provides basic styling for the gallery.
- The `index.php` file displays a welcome message on the home page.
    - `ob_start()` and `ob_get_clean()` are used to capture the output generated within the PHP block and assign it to the $content variable.
    - The $page_title and $content variables are used to customize the title and content of the page. These variables are then included in the template using PHP echo statements.


## 2. Create File Upload Form

Next, we will create a form to allow users to upload images to the gallery.

1. Create a file named `upload.php` with the following content:

```php
<?php
$page_title = "Upload";
ob_start();
?>

<h2>Upload Image</h2>
<form class="file-upload" method="post" enctype="multipart/form-data">
    <input type="file" name="uploaded_file" accept=".jpg, .jpeg, .png, .gif">
    <button type="submit">Upload</button>
</form>

<?php
$content = ob_get_clean();
include "template.php";
?>
```

Explanation:
- The `upload.php` file contains a form for users to upload images to the gallery.
    - The form uses the `enctype="multipart/form-data"` attribute to allow file uploads.
    - The `accept` attribute specifies the file types that can be uploaded (in this case, images with extensions .jpg, .jpeg, .png, .gif).
    - The form does not have an `action` attribute. When the `action` attribute is omitted, the form will submit to the same page by default. 
- The uploaded file will be processed in the next steps.

2. Style the upload form by adding CSS to the `styles.css` file.


## 3. Handle File Upload

Now, we will write a PHP script to handle the file upload process.

1. Create a directory named `uploads` to store the uploaded images.

2. Modify the `upload.php` file to handle the file upload process. Add the following code to the start of `upload.php`:

```php
$response = "";


if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $targetDir = "uploads/";
    $targetFile = $targetDir . basename($_FILES["uploaded_file"]["name"]);

    if (move_uploaded_file($_FILES["uploaded_file"]["tmp_name"], $targetFile)) {
        $response = "The file " . htmlspecialchars(basename($_FILES["uploaded_file"]["name"])) . " has been uploaded.";
    } else {
        $response = "Sorry, there was an error uploading your file.";
    }
}
```

3. Update the form in `upload.php` to display the upload response message. Add the following code after the form:

```php
<p><?php if (!empty($response)) echo $response; ?></p>
```

Explanation:

- The PHP script checks if the form has been submitted using the `$_SERVER["REQUEST_METHOD"]` variable.
- The uploaded file is moved to the `uploads/` directory using the `move_uploaded_file()` function.
- If the file upload is successful, a success message is displayed. Otherwise, an error message is displayed.
- The response message is displayed below the upload form.


## 4. Generate Thumbnail Images

To enhance the gallery display, we will generate thumbnail images for each uploaded image.

1. Create a directory named `thumbnails` to store the thumbnail images.

2. Create a new file named `generate_thumbnail.php` with the following content:

```php
<?php
function createSquareThumbnail($sourceFile, $thumbnailFile, $thumbnailSize = 200) {
    // Get the image type and dimensions
    list($width, $height, $imageType) = getimagesize($sourceFile);

    // Create an image resource from the source file based on its type
    switch ($imageType) {
        case IMAGETYPE_JPEG:
            $sourceImage = imagecreatefromjpeg($sourceFile);
            break;
        case IMAGETYPE_PNG:
            $sourceImage = imagecreatefrompng($sourceFile);
            break;
        case IMAGETYPE_GIF:
            $sourceImage = imagecreatefromgif($sourceFile);
            break;
        default:
            throw new Exception("Unsupported image type.");
    }

    // Determine the size of the square crop
    $minSize = min($width, $height);
    $cropX = ($width - $minSize) / 2;  // Center crop horizontally
    $cropY = ($height - $minSize) / 2; // Center crop vertically

    // Create a blank square thumbnail
    $thumbnail = imagecreatetruecolor($thumbnailSize, $thumbnailSize);

    // Preserve transparency for PNG and GIF
    if ($imageType == IMAGETYPE_PNG || $imageType == IMAGETYPE_GIF) {
        imagecolortransparent($thumbnail, imagecolorallocatealpha($thumbnail, 0, 0, 0, 127));
        imagealphablending($thumbnail, false);
        imagesavealpha($thumbnail, true);
    }

    // Copy and resize the cropped portion of the source image into the thumbnail
    imagecopyresampled(
        $thumbnail, $sourceImage,
        0, 0, $cropX, $cropY, // Destination and source positions
        $thumbnailSize, $thumbnailSize, // Destination size
        $minSize, $minSize // Source size
    );

    // Save the thumbnail to the specified file
    switch ($imageType) {
        case IMAGETYPE_JPEG:
            imagejpeg($thumbnail, $thumbnailFile);
            break;
        case IMAGETYPE_PNG:
            imagepng($thumbnail, $thumbnailFile);
            break;
        case IMAGETYPE_GIF:
            imagegif($thumbnail, $thumbnailFile);
            break;
    }

    // Free memory
    imagedestroy($sourceImage);
    imagedestroy($thumbnail);
}

?>
```

3. Include the `generate_thumbnail.php` file in the `upload.php` file to use the `createSquareThumbnail()` function.

```php
include "generate_thumbnail.php";
```

4. Modify the file upload handling section in `upload.php` to generate a thumbnail image (after moving the uploaded file):

```php
// Generate thumbnail image
$thumbnailDir = "thumbnails/";
$thumbnailFile = $thumbnailDir . basename($_FILES["uploaded_file"]["name"]);

createSquareThumbnail($targetFile, $thumbnailFile);
```

Explanation:

- The `generate_thumbnail.php` file contains a function `createSquareThumbnail()` that generates a square thumbnail image from the source image.
- The function takes the source file path, thumbnail file path, and optional thumbnail size as parameters.
- The function determines the image type and dimensions, creates a blank square thumbnail, and copies the cropped portion of the source image into the thumbnail.
- The thumbnail is saved to the specified file path based on the image type.
- The function supports JPEG, PNG, and GIF image types.
- The `createSquareThumbnail()` function is used in the `upload.php` file to generate a thumbnail image for each uploaded image.


## 5. Display Gallery Images

Finally, we will create a gallery page to display the uploaded images and their thumbnails.

1. Create a file named `gallery.php` with the following content:

```php
<?php
$page_title = "Gallery";
ob_start();
?>

<h2>Photo Gallery</h2>
<div class="gallery">
    <?php
    $imagesDir = "uploads/";
    $thumbnailsDir = "thumbnails/";

    $images = glob($imagesDir . "*.{jpg,jpeg,png,gif}", GLOB_BRACE);

    foreach ($images as $image) {
        $thumbnail = $thumbnailsDir . basename($image);
        echo "<a href='$image' target='_blank'><img src='$thumbnail' alt='Gallery Image'></a>";
    }
    ?>
</div>

<?php
$content = ob_get_clean();
include "template.php";
?>
```

2. Style the gallery images by adding CSS to the `styles.css` file.

Explanation:

- The `gallery.php` file displays the uploaded images and their thumbnails in a gallery format.
- The `uploads/` directory contains the original images, while the `thumbnails/` directory contains the generated thumbnail images.
- The `glob()` function is used to retrieve all image files (with extensions .jpg, .jpeg, .png, .gif) from the `uploads/` directory.
- For each image, an anchor tag (`<a>`) is created with the original image as the target and the thumbnail image as the source.
- Clicking on a thumbnail will open the original image in a new tab.


## 6. Further Enhancements

To further enhance the photo gallery, you can consider implementing the following features:

- Use a modal or lightbox to display larger versions of the images when clicked instead of opening in a new tab.
- Add pagination to the gallery if there are a large number of images.
- Implement image sorting options (e.g., by date, name, size).
- Allow users to delete images from the gallery.
- Add image captions or descriptions to the gallery display.
