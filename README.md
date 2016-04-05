# Laravel 5 in openShift Without Mcrypt

By this repo, we can install Laravel 5 in the openshift gear without php mcrypt extension.

This package uses a trick to fool composer into thinking that your system has the PHP Mcrypt extension installed. Allowing you to install Laravel on shared hosts or systems where Mcrypt is not present.
In addition it provides two service providers `NoEncryptionServiceProvider` and `OpensslEncryptionServiceProvider` for use as an alternative to `Illuminate\Encryption\EncryptionServiceProvider`

## Installation

### Installing dependencies on the wrong environment is now possible

The new --ignore-platform-reqs flag for the install and update commands lets you install dependencies even if you have the wrong php version or are missing one of the required php extensions. It's not really recommended but it can be useful sometimes if you want to run composer outside a VM for example and you only have the right extensions installed in the VM where you run the code. 

Upload Laravel 5 to your openshift gear, and run

```
rhc ssh app
cd app-root/runtime/repo
composer install --ignore-platform-reqs
```

Install via composer to an exisiting working Laravel project.

```
composer require thomaswelton/laravel-mcrypt-faker --ignore-platform-reqs
```

If you are unable to install Laravel via composer due to no having Mcrypt installed then you will need to add this package manually.
Download the latest Laravel source and edit the `composer.json` so your require block looks as follows.

```
"require": {
	"thomaswelton/laravel-mcrypt-faker": "1.0.*",
	"laravel/framework": "5.0.*"
}
```

The run `composer install` which will install this package along with the laravel framework.


## Updating the Encryption Service Provider

In your `config/app.php` file remove `Illuminate\Encryption\EncryptionServiceProvider` from the `providers` array and replace it with either `Thomaswelton\LaravelMcryptFaker\NoEncryptionServiceProvider` or `Thomaswelton\LaravelMcryptFaker\OpensslEncryptionServiceProvider`

Also update the `cipher` in `config/app.php` and set it to `null` as the cipher value `MCRYPT_RIJNDAEL_128` is a constant that would not be defined without mcrypt

**WARNING** The NoEncryptionServiceProvider, as the name suggests, provides no encryption for your application... at all. This should not be used in a production website. And even though the `OpensslEncryptionServiceProvider` provides encryption using the defuse/php-encryption package I personally can not attest to how cryptographically secure it's implementation is, even thought it "Works for me" 


### OpensslEncryption Key

To use the OpensslEncryptionServiceProvider your app secret key needs to be updated. This can be done by running `php artisan key:generate-openssl`
