## Run single cron job on multiple server 

### Create EC2 Instance in AWS

First, open the Amazon EC2 console at https://console.aws.amazon.com/ec2/ and click on “Running Instances”. 

- Click on Launch Instance.

Choose which type of Virtual Machine you would like to use. The Amazon Machine Images are pre-configured virtual machines that serve as a template for your instance. 
 - For our application select "Ubuntu 20.04 (64bit)" Instance.

Next, choose an instance type to set the storage and size of your instance. 
 - T2.micro is set as the type by default and eligible for Amazon’s free tier. 


Next, click on Review and Launch to let the AWS wizard set up the default configurations for you.

Next, review your instance and modify security groups, In that
 - Add "http" as Type with port 80 and choose source "0.0.0.0/0" and click "Launch"

You will have to specify a key pair. If you already have an AWS key-pair, you can select to use that one. If not, you will select Create a New Key Pair. Enter any name for the key pair. Then, click Download Key Pair. This will be the only time you will be able to download this key pair. Move it to some secure location, or somewhere you will not delete it. You need this key-pair to allow you to enter your instance. Consider it a password. Now, launch your instance!

Now that your instance is launched, next we have to connect to the instance via SSH. On the EC2 instances dashboard, click the button next to your instance name, and click on Actions. A dropdown menu should appear under Actions, and click on Connect.

Open Terminal and go to the folder at which your key-pair is stored. Run the chmod command to change permissions on your key-pair. Then, run the ssh command to enter your instance. You will need to rerun the chmod command to modify the permissions on your key-pair every time you restart your computer in order to enter your instance.

### Installing Dependencies for Laravel
In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to obtain this software.

#### Installing the Nginx Web Server

Since this is our first time using apt for this session, start off by updating your server’s package index. Following that, you can use apt install to get Nginx installed:

```sh
sudo apt update
sudo apt install nginx
```

When prompted, enter Y to confirm that you want to install Nginx. Once the installation is finished, the Nginx web server will be active and running on your Ubuntu 20.04 server.

If you do not have a domain name pointed at your server and you do not know your server’s public IP address, you can find it by running the following command:

```sh
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
This will print out a few IP addresses. You can try each of them in turn in your web browser.
Type the address that you receive in your web browser and it will take you to Nginx’s default landing page:

```sh
http://server_domain_or_IP
```

#### Installing PHP
You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

To install the php-fpm and php-mysql packages, run:

```sh
sudo apt install php-fpm php-mysql

```

When prompted, type Y and ENTER to confirm installation.
You now have your PHP components installed. Next, you’ll configure Nginx to use them.

#### Configuring Nginx to Use the PHP Processor
open configuration file in Nginx’s sites-enabled directory using your preferred command-line editor. Here, we’ll use vim:

```sh
sudo vim /etc/nginx/sites-enabled/default
```

Paste in the following bare-bones configuration:

```sh
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }
}
```
When you’re done editing, save and close the file. 

This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:

```sh
sudo nginx -t
```

If any errors are reported, go back to your configuration file to review its contents before continuing.

When you are ready, reload Nginx to apply the changes:

```sh
sudo service nginx restart
```

Your LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.

#### Testing PHP with Nginx
Your LEMP stack should now be completely set up. You can test it to validate that Nginx can correctly hand .php files off to your PHP processor.

You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:

```sh
vim /var/www/html/info.php
```

Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:

```sh
<?php
phpinfo();
```
When you are finished, save and close the file. 
You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:

```sh
http://server_domain_or_IP/info.php
```
You will see a web page containing detailed information about your server:

After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:

```sh
sudo rm /var/www/html/info.php
```

#### Installing Additional Dependencies
In addition to dependencies that should be already included within your Ubuntu 20.04 system, such as git and curl, Composer requires php-cli in order to execute PHP scripts in the command line, and unzip to extract zipped archives. We’ll install these dependencies now.

First, update the package manager cache by running:

```sh
sudo apt update
```
Next, run the following command to install the required packages:

```sh
sudo apt install php-cli unzip
```
You will be prompted to confirm installation by typing Y and then ENTER.

Once the prerequisites are installed, you can proceed to installing Composer.

#### Downloading and Installing Composer
Composer provides an installer script written in PHP. We’ll download it, verify that it’s not corrupted, and then use it to install Composer.

Make sure you’re in your home directory, then retrieve the installer using curl:

```sh
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
```

Next, we’ll verify that the downloaded installer matches the SHA-384 hash for the latest installer found on the Composer Public Keys / Signatures page. To facilitate the verification step, you can use the following command to programmatically obtain the latest hash from the Composer page and store it in a shell variable:

```sh
HASH=`curl -sS https://composer.github.io/installer.sig`
```
If you want to verify the obtained value, you can run:

```sh
echo $HASH
```
output:
```sh
e0012edf3e80b6978849f5eff0d4b4e4c79ff1609dd1e613307e16318854d24ae64f26d17af3ef0bf7cfb710ca74755a
```
Now execute the following PHP code, as provided in the Composer download page, to verify that the installation script is safe to run:

```sh
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```
You’ll see the following output:

```sh
Installer verified
```
To install composer globally, use the following command which will download and install Composer as a system-wide command named composer, under /usr/local/bin:
```sh
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
You’ll see output similar to this:

```sh
All settings correct for using Composer
Downloading...

Composer (version 1.10.5) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer
```
To test your installation, run:

```sh
composer
```
Output:
```sh
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.10.5 2020-04-10 11:44:22

Usage:
  command [options] [arguments]

Options:
  -h, --help                     Display this help message
  -q, --quiet                    Do not output any message
  -V, --version                  Display this application version
      --ansi                     Force ANSI output
      --no-ansi                  Disable ANSI output
  -n, --no-interaction           Do not ask any interactive question
      --profile                  Display timing and memory usage information
      --no-plugins               Whether to disable plugins.
  -d, --working-dir=WORKING-DIR  If specified, use the given directory as working directory.
      --no-cache                 Prevent use of the cache
  -v|vv|vvv, --verbose           Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
...
```
This verifies that Composer was successfully installed on your system and is available system-wide.

#### Installing Required PHP modules for Laravel
Before you can install Laravel, you need to install a few PHP modules that are required by the framework. We’ll use apt to install the php-mbstring, php-xml and php-bcmath PHP modules. These PHP extensions provide extra support for dealing with character encoding, XML and precision mathematics.

update the package manager cache by running:

```sh
sudo apt update
```
Now you can install the required packages with:

```sh
sudo apt install php-mbstring php-xml php-bcmath
```

Your system is now ready to execute Laravel’s installation via Composer, but before doing so, you’ll need a database for your application.

### Creating a New Laravel Application

You will now create a new Laravel application using the composer create-project command. This Composer command is typically used to bootstrap new applications based on existing frameworks and content management systems.

First, go to your user’s home directory:

```sh
cd ~
```

The following command will create a new my_app_name directory containing a barebones Laravel application based on default settings:

```sh
composer create-project --prefer-dist laravel/laravel my_app_name
```
You will see output similar to this:

```sh
Installing laravel/laravel (v5.8.17)
  - Installing laravel/laravel (v5.8.17): Downloading (100%)         
Created project in my_app_name
> @php -r "file_exists('.env') || copy('.env.example', '.env');"
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 80 installs, 0 updates, 0 removals
  - Installing symfony/polyfill-ctype (v1.11.0): Downloading (100%)         
  - Installing phpoption/phpoption (1.5.0): Downloading (100%)         
  - Installing vlucas/phpdotenv (v3.4.0): Downloading (100%)         
  - Installing symfony/css-selector (v4.3.2): Downloading (100%)     
...
```

When the installation is finished, access the application’s directory and run Laravel’s artisan command to verify that all components were successfully installed:

```sh
cd my_app_name
php artisan
```

You’ll see output similar to this:

```sh
Laravel Framework 8.11.0

Usage:
  command [options] [arguments]

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
      --env[=ENV]       The environment the command should run under
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
...
```

This output confirms that the application files are in place, and the Laravel command-line tools are working as expected. However, we still need to configure the application to set up the database and a few other details.

#### Configuring Laravel
The Laravel configuration files are located in a directory called config, inside the application’s root directory. Additionally, when you install Laravel with Composer, it creates an environment file. This file contains settings that are specific to the current environment the application is running, and will take precedence over the values set in regular configuration files located at the config directory. Each installation on a new environment requires a tailored environment file to define things such as database connection settings, debug options, application URL, among other items that may vary depending on which environment the application is running.

> Warning: The environment configuration file contains sensitive information about your server, including database credentials and security keys. For that reason, you should never share this file publicly.

We’ll now edit the .env file to customize the configuration options for the current application environment.

Open the .env file using your command line editor of choice.

```sh
vim .env
```

Even though there are many configuration variables in this file, you don’t need to set up all of them now. The following list contains an overview of the variables that require immediate attention:

* APP_NAME: Application name, used for notifications and messages.
* APP_ENV: Current application environment.
* APP_KEY: Used for generating salts and hashes, this unique key is automatically created when installing Laravel via Composer, so you don’t need to change it.
* APP_DEBUG: Whether or not to show debug information at client side.
* APP_URL: Base URL for the application, used for generating application links.
* DB_DATABASE: Database name.
* DB_USERNAME: Username to connect to the database.
* DB_PASSWORD: Password to connect to the database.

The following .env file sets up our example application for development:

```sh
APP_NAME=My App Name
APP_ENV=production
APP_KEY=APPLICATION_UNIQUE_KEY_DONT_COPY
APP_DEBUG=true
APP_URL=http://domain_or_IP

LOG_CHANNEL=stack

# We will update the below database details after this setup
DB_CONNECTION=mysql
DB_HOST=DATABASE_HOST
DB_PORT=3306
DB_DATABASE=DATABASE_NAME
DB_USERNAME=DATABASE_USERNAME
DB_PASSWORD=DATABASE_PASSWORD
```

Adjust your variables accordingly. When you are done editing, save and close the file to keep your changes. If you’re using nano, you can do that with CTRL+X, then Y and Enter to confirm.

Your Laravel application is now set up, but we still need to configure the web server in order to be able to access it from a browser. In the next step, we’ll configure Nginx to serve your Laravel application.

#### Setting Up Nginx

We have installed Laravel on a local folder of your remote user’s home directory, and while this works well for local development environments, it’s not a recommended practice for web servers that are open to the public internet. We’ll move the application folder to /var/www, which is the usual location for web applications running on Nginx.

First, use the mv command to move the application folder with all its contents to /var/www/my_app_name:

```sh
sudo mv ~/my_app_name /var/www/my_app_name
```

Now we need to give the web server user write access to the storage and cache folders, where Laravel stores application-generated files:

```sh
sudo chown -R www-data.www-data /var/www/my_app_name/storage
sudo chown -R www-data.www-data /var/www/my_app_name/bootstrap/cache
```

The application files are now in order, but we still need to configure Nginx to serve the content. To do this, we’ll create a new virtual host configuration file at /etc/nginx/sites-available:

```sh
sudo vim /etc/nginx/sites-available/my_app_name
```

The following configuration file contains the recommended settings for Laravel applications on Nginx:

```sh
server {
    listen 80;
    server_name _;
    root /var/www/my_app_name/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Copy this content to your /etc/nginx/sites-enabled/default file and, if necessary, adjust the highlighted values to align with your own configuration. Save and close the file when you’re done editing.

To confirm that the configuration doesn’t contain any syntax errors, you can use:

```sh
sudo nginx -t
```

You should see output like this:
```sh
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

To apply the changes, restart Nginx with:

```sh
sudo systemctl reload nginx
```
Now go to your browser and access the application using the server’s domain name or IP address, as defined by the server_name directive in your configuration file:

```sh
http://server_domain_or_IP
```

You will see the Laravel default landing page.

That confirms your Nginx server is properly configured to serve Laravel. From this point, you can start building up your application on top of the skeleton provided by the default installation.

### Creating Database using AWS RDS
Open the Amazon RDS console at https://console.aws.amazon.com/rds/ and 

* click on “Create Database”.

* Select the MySQL engine and click Next. Select the Dev/Test use case which keeps us within the RDS Free Usage Tier.

* The next section is where we will specify the Database details. Select the t2.micro DB instance type, then scroll down to the Settings row at the bottom.

* Set the DB instance identifier, the Master username, the Master Password and the Confirm Password

* Cick on "Create database"

Your DB Instance is now being created.  Click View Your DB Instances.

>Note: Depending on the DB instance class and storage allocated, it could take several minutes for the new DB instance to become available.

The new DB instance appears in the list of DB instances on the RDS console. The DB instance will have a status of creating until the DB instance is created and ready for use.  When the state changes to available, you can connect to a database on the DB instance. 

Once the DB instance is created, Choose "modify" option and then select ""

Click on the DB instance you have created and copy the Endpoint in Connectivity and security, you need to use it in MySQL connection(Host name).

* Use the "end point" as a host and use the username and password which created during the Database creating process, in the laravel application's ".env" file.

#### Configuring Laravel .env File
Now log back into our EC2 Instance via SSH and navigate to the project directory

```sh
cd /var/www/my_app_name
```

Open the environment file using your text editor,

```sh
vim .env
```
And update the database details in the .env file in the below fields,

```sh
...
DB_CONNECTION=mysql
DB_HOST=<RDS ENDPOINT>
DB_PORT=3306
DB_DATABASE=<RDS DATABASE NAME>
DB_USERNAME=<RDS DATABASE USERNAME>
DB_PASSWORD=<RDS DATABASE PASSWORD>
...
```
Once done, save and close the file

Now we need to give Nginx write access to the storage folder, else Laravel will throw a write permission error.

```sh
sudo chmod -R 777 /var/www/my_app_name/storage
```

#### Creating Cache Tables
We are using the database cache driver, you will need to setup a table to contain the cache items.

Artisan command to generate a migration with the proper schema. Run the following command in the project directory

```sh
php artisan cache:table
php artisan migrate`
```

Now the cache table has been created in the database. 

### Changing Cache Configuration

We need to tell the application to use our database cache in order to serve commands to run in a single server

For that open cache config file located in "/my_app_name/config/cache.php"
In that change the particular "CACHE_DRIVER" config to "database" as below

```sh
 'default' => env('CACHE_DRIVER', 'database'),
 ```
 
 Save and close the file.
 
 Next, open the ".env" file and change the "CACHE_DRIVER" config to "database" as below
 
 ```sh
 CACHE_DRIVER=database
 ```
 
 Save and close the file.
 Now we have setup all the cache configuration to database in our application.

#### Create Console Commands 

To create a new command, you may use the "make:command" Artisan command. This command will create a new command class in the "app/Console/Commands" directory. Don't worry if this directory does not exist in your application - it will be created the first time you run the "make:command" Artisan command:

```sh
php artisan make:command CommandName
```

The console commands file will be created in the location 
"/my_app_name/app/Console/Commands/CommandName.php"

Open the file and replace the contents below,

```sh
<?php

namespace App\Console\Commands;
use Illuminate\Support\Facades\DB;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Cache;

class CommandName extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'command:name';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        Cache::lock("update-visited-1")->get(function () {
            // Your command function code goes here
        });
    }
}
```

Save and close the file.

Now run the console commands to check if its working properly by using the command below,

```sh
php artisan command:name
```

After this, Add this command in "kernal.php". To do so, open "/my_app_name/app/Console/Kernel.php" and add the command as below,

```sh
protected function schedule(Schedule $schedule)
    {
        $schedule->command('command:name')->everyMinute()->onOneServer();
    }
```

save and close the file.

To check the above condition, run the schedule job by running the below command,

```sh
php artisan schedule:run
```

It should run the functions we created in the console commands.
Once it is verified, Last step is to add this scheduler in the crontab.

Open crontab by using,

```sh
crontab -e
```

In that file add the following line to add our scheduler in the crontab,

```sh
* * * * * cd /var/www/my_app_name && php artisan schedule:run >> /dev/null 2>&1
```

That's all. Viola !
We did this. 

We need to take a AMI Image of this instance and launch the AMI as a new instance and check all "Security Groups" to allow "http" with port "80" to server our application and launch the instance. 

Now we have the multiple instance with single cron running. 
