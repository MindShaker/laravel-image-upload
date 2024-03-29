# Laravel Image Upload

This package makes image upload easy. It uses [Intervention Image](https://image.intervention.io/v3) under the hood. 

## Installation
Require this package, with [Composer](https://getcomposer.org), in the root directory of your project.

```bash
$ composer require mindshaker/image-upload
```

## Configuration
There are some default configurations

```php
return [
    'public_path' => 'uploads',
    'random_name' => true,
];
```

You can change this values by publishing this configuration file

```bash
$ php artisan vendor:publish --provider="Mindshaker\\ImageUpload\\ImageUploadServiceProvider"
```

If you want to make your images private (blocked by auth for example), you'll need to add a new disk in `config/filesystems.php` and put this in the disks array
```php
'private' => [
    'driver' => 'local',
    'root' => storage_path('app/upload_folder'),
    'url' => env('APP_URL') . '/storage',
    'visibility' => 'private',
    'throw' => false,
],
```

## Usage
There are only a few methods, you can call the `upload` method to upload a image that will be edited to the given dimensions or cropped. The image will be saved to the public_path set in the configurations and be given a random name or the name of the original file. 

The upload function returns the image name with additional path if set (see bellow)

```php
use Mindshaker\ImageUpload\Facades\ImageUpload;

ImageUpload::upload($image, $width = null, $height = null, $crop = false, $private = false);
```

To add an additional path
```php
ImageUpload::path("additional_path");
```

To give specific name to the uploaded file, if set it will ignore the configuration `random_name`

```php
ImageUpload::name("image_name");
```

To define the format of the image uploaded (e.g. png, jpg, webp, etc..)

```php
ImageUpload::format("jpg");
```

To define the quality of the image, default 75.

```php
ImageUpload::quality(75);
```

You can delete images by calling the method `delete($image, $private = false)` where `$image` is the name returned from the `upload()` method

```php
$image_name = ImageUpload::upload($image, 1920);

ImageUpload::delete($image_name);
```

### Basic Examples

```php
use Mindshaker\ImageUpload\Facades\ImageUpload;

$image = $request->file('image');

//Simple upload ($width or $height needs to be specified)
//It will resize the image to 1920px. It won't upscale
ImageUpload::upload($image, 1920);

//Crop the image to the given dimensions
ImageUpload::upload($image, 512, 512, true);

//Change the format of the image
ImageUpload::format("webp")->upload($image, 1920);

//Add aditional path and change name
ImageUpload::path("posts/{id}")->name("post_name")->format("webp")->upload($image, 1920, null);
//returns something like "posts/1/post_name.webp"
```

## Adavanced Usage

If you want more than just resizing and cropping the images, you can call the method `manager()` and edit the image like it's an [Intervention Image](https://image.intervention.io/v3). This is a substitute to the upload method, all the other methods still work (Like `path()`, `name()` and `format()`). After editing, to upload the image you can call the method `save()` to upload the image to the specified path and format.

```php
ImageUpload::manager($image);
```

### Example

```php
use Mindshaker\ImageUpload\Facades\ImageUpload;

$image = $request->file('image');

ImageUpload::manager($image)->pad(512, 512, 'ccc')->format("png")->save();
```