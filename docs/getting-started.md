Getting Started
===============

Requirements
-------
 - PHP 7.0+
 - Laravel 5.4+

Installation
------------

It is recommended to install the DMS via [Composer](https://getcomposer.org/) in a fresh Laravel project.

 1. Create a new Laravel project [More details](https://laravel.com/docs/5.4/installation)
    
    ```
    composer create-project --prefer-dist laravel/laravel your-project-name
    cd your-project-name
    ```
    
 2. Edit the `.env` file and enter the correct database credentials.
 
    ```
    # Enter the correct database credentails
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=homestead
    DB_USERNAME=homestead
    DB_PASSWORD=secret
    ```
    
 2. Install the latest version of DMS via Composer
 
    ```
    composer require dms/web.laravel
    ```
    
 3. Add the service provider your `config/app.php`
    
    ```php
    /*
     * Package Service Providers...
     */
    Dms\Web\Laravel\DmsServiceProvider::class,
    ```
  
 4. Run the DMS installation command
     
    ```
    php artisan dms:install
    ```

 5. Visit `http://your-app-domain/dms` to view the backend of your new project
    
    You can login with the default user account. **U:** admin **P:** admin.
    
    ![Login](/resources/images/cms/login-1.jpg)