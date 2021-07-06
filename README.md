<div align="center">
	<h1> Note </h1>
   This Project is from <a href="https://github.com/nao-pon"> nao-pon/ </a>
	<p>I cloned it here to apply some changes for an urgent project, due to the impossibility to edit vendor files directly</p>
	<p>so all efforts go to nao-pon/</p>
	<a href="https://github.com/nao-pon/flysystem-google-drive">check the original repo here</a>
</div>

<br>
# Flysystem Adapter for Google Drive

[![Author](https://img.shields.io/badge/author-nao--pon%20hypweb-blue.svg?style=flat)](http://xoops.hypweb.net/)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)


## Installation
Steps won't be the same as the original repo.


```bash
composer require spatie/laravel-backup 7.3.3
```
```bash
php artisan vendor:publish --provider="Spatie\Backup\BackupServiceProvider"
```
 - Then in config/backup.php add the following:
```
'disks' => [
    'google',                
    'local',             
],
```

```bash
composer require jervi-sir/flysystem-gdrive-api:~1.1
```

```bash
php artisan make:provider GoogleDriveServiceProvider
```

- In app/Providers/GoogleDriveServiceProvider.php, also declare it in config/app.php file.
```
public function boot()
{
    \Storage::extend('google', function ($app, $config) {
        $client = new \Google_Client();
        $client->setClientId($config['clientId']);
        $client->setClientSecret($config['clientSecret']);
        $client->refreshToken($config['refreshToken']);
        $service = new \Google_Service_Drive($client);
        $adapter = new \Hypweb\Flysystem\GoogleDrive\GoogleDriveAdapter($service, $config['folderId']);

        return new \League\Flysystem\Filesystem($adapter);
    });
}
```
- Declare google provider in config/app.php
```
App\Providers\GoogleDriveServiceProvider::class,
```
- In config/filesystem.php
```
return [
  
    // ...
    
    'disks' => [
        
        // ...
        
        'google' => [
            'driver' => 'google',
            'clientId' => env('GOOGLE_DRIVE_CLIENT_ID'),
            'clientSecret' => env('GOOGLE_DRIVE_CLIENT_SECRET'),
            'refreshToken' => env('GOOGLE_DRIVE_REFRESH_TOKEN'),
            'folderId' => env('GOOGLE_DRIVE_FOLDER_ID'),
        ],
        
        // ...
        
    ],
    
    // ...
];
```
- In .env
```
GOOGLE_DRIVE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_DRIVE_CLIENT_SECRET=xxx
GOOGLE_DRIVE_REFRESH_TOKEN=xxx
GOOGLE_DRIVE_FOLDER_ID=null
```

## Get Google credentials

[Check this article](https://qirolab.com/posts/how-to-setup-laravel-backup-on-google-drive-1607368130) of How To Setup Laravel Backup On Google Drive?

Then execute following bash 
```
php artisan backup:run
```

## Usage
- In a controller
```
//Create a disk
$disk = Storage::disk('google');
//Create Image Object (Image Intervention package must be installed)
$img = Image::make($image)->orientate()->encode('jpg', 50);
$img->encode();
//Upload file 
$disk->put($file_name, $img->getEncoded());
//get the Id
$pic_id = $disk->getDriver()->getAdapter()->getCacheFileObjectsByName()->id;
```


## Remark
Notice that in previous code I called the function getCacheFileObjectsByName(), this is the only change I needed for this package, to get access to the private variable CacheFileObjectsByName, so I can get uploaded file's id
- here is the change 
In GoogleDriveAdapter.php, I added
```
public function getCacheFileObjectsByName()
{
    return array_values($this->cacheFileObjectsByName)[0];
}
```


## Credit

- [How to setup Laravel backup on google drive](https://qirolab.com/posts/how-to-setup-laravel-backup-on-google-drive-1607368130)

- [Flysystem Adapter for Google Drive](https://github.com/nao-pon/flysystem-google-drive)

