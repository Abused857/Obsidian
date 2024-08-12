
composer require google/cloud-storage

composer require spatie/laravel-google-cloud-storage

example filesystem en config
```php
'gcs' => [
    'driver' => 'gcs',
    'key_file_path' => env('GOOGLE_CLOUD_KEY_FILE', base_path('laravel-gcs.json')),
    'project_id' => env('GOOGLE_CLOUD_PROJECT_ID', 'your-project-id'),
    'bucket' => env('GOOGLE_CLOUD_STORAGE_BUCKET', 'your-bucket'),
    'path_prefix' => env('GOOGLE_CLOUD_STORAGE_PATH_PREFIX', ''),
    'storage_api_uri' => env('GOOGLE_CLOUD_STORAGE_API_URI', null),
    'apiEndpoint' => env('GOOGLE_CLOUD_STORAGE_API_ENDPOINT', null),
    'visibility' => 'private',
    'visibility_handler' => null,
    'metadata' => ['cacheControl'=> 'public,max-age=86400'],
    'throw' => true,
],
```

.env 
```php
  

FILESYSTEM_DRIVER=gcs

  

GOOGLE_CLOUD_PROJECT_ID=

GOOGLE_CLOUD_KEY_FILE=

GOOGLE_CLOUD_BUCKET=

GOOGLE_CLOUD_PATH_PREFIX=

GOOGLE_CLOUD_STORAGE_API_URI=

```

crear un registro de file:

```php
  

                    $file = File::create([

                        'name' => $uploadedFile->getClientOriginalName(),

                        'size' => $uploadedFile->getSize(),

                        'mimetype' => $uploadedFile->getMimeType(),

                        'url' => $fileUrl,

                        'type_id' => 1,

                    ]);

```


ejemplo de guardar una file 

```php
  
   if ($request->hasFile('file')) {

                    $uploadedFile = $request->file('file');

                    $path = Storage::disk('gcs')->put('files', $uploadedFile);

                    $fileUrl = Storage::disk('gcs')->url($path);

                    $file = File::create([

                        'name' => $uploadedFile->getClientOriginalName(),

                        'size' => $uploadedFile->getSize(),

                        'mimetype' => $uploadedFile->getMimeType(),

                        'url' => $fileUrl,

                        'type_id' => 1,

                    ]);

```

ejemplo de guardar un array de files

```php
  
$files = $request->file('files');
$fileUrls = [];

if ($files) {
    if (is_array($files)) {
        foreach ($files as $file) {
            if ($file instanceof \Illuminate\Http\UploadedFile) {
                $path = Storage::disk('gcs')->putFile('files', $file);
                $fileUrl = Storage::disk('gcs')->url($path);
                $fileUrls[] = $fileUrl;
            }
        }
    } elseif ($files instanceof \Illuminate\Http\UploadedFile) {
        $path = Storage::disk('gcs')->putFile('files', $files);
        $fileUrl = Storage::disk('gcs')->url($path);
        $fileUrls[] = $fileUrl;
    } else {
        return response()->json(['error' => 'Invalid file format.'], 400);
    }
} else {
    return response()->json(['message' => 'No files uploaded.'], 400);
}

return response()->json(['file_urls' => $fileUrls]);


```

file controller no se si va hacer falta aun

```php
  
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class FileUploadController extends Controller
{
    public function uploadFileToCloud(Request $request)
    {
        try {
            $file = $request->file('file');
            $file_name = time() . '_' . $file->getClientOriginalName();
            $storeFile = $file->storeAs("test", $file_name, "gcs");
            $disk = Storage::disk('gcs');
            $fetchFile = $disk->url($storeFile);
        } catch(\UnableToWriteFile|UnableToSetVisibility $e) {
            throw_if($this->throwsExceptions(), $e);
        return false;
        }

        return response()->json([
                'data' => $fetchFile,
        ], 201);
    }
}


```

crear la ruta

```php
  
Route::post(
    "/upload-file", 
    [App\Http\Controllers\FileUploadController::class, "uploadFileToCloud"]
);


```

[[Laravel instalación]] [[Controller]] 