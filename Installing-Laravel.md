Installing Laravel
==================

This guide is an expanded version of the [Laravel 5.3 Homestead guide](http://laravel.com/docs/5.3/homestead) with modifications specific to Windows 7-10.
<<<<<<< HEAD
=======

If you're on macOS, you can check out [Laravel Valet](https://laravel.com/docs/5.3/valet) if your needs are simple, or follow the above Homestead guide.
>>>>>>> a64be4f1e445bc790f1f0f2d692849d920956633

Assumptions
-----------

* The term **dev_root** means your local development directory, i.e. the root directory in which you keep your development projects. For example, `c:\users\me\dev\`


1. Laravel Installation with Homestead
------------------------------
This section explains the **recommended** way to use Laravel. See later in this document for the steps to install Laravel natively on your Windows PC. 

This method uses a VirtualBox VM running Ubuntu, called _Homestead_, to encompass all of your PHP development components, which has the following benefits:

1. Your development environment is not tied to your PC - it is portable in two senses:
  + you can copy the VM to another PC using `vagrant package` [vagrant package docs](http://docs.vagrantup.com/v2/cli/package.html)`
  + assuming you haven't modified Homestead, or you've encapsulated the changes using a provisioning tool like Ansible, you can `git clone` and then `vagrant up` to redeploy the dev environment to another PC
2. Everyone in the team will have an identical PHP development environment - even if Windows is a different version, or they have different configurations, it won't affect the dev environment
3. PHP on Windows is not the easiest thing to manage, so keeping it on Linux where it likes to be is going to make your life a little bit more simple
4. Homestead is configured using [Vagrant](https://www.vagrantup.com/), which uses a scripted approach to configuring the VM. Vagrant can also be used to configure EC2 and other cloud provider instances. This means you can be sure that your development environment matches your Production environment.
5. It's easy to have completely different configurations (e.g. php 5.6 with Apache, or php FPM 7.0 with nginx) available on your machine without having to worry about conflicts.

The Homestead VM gives you the following components pre-installed:
* Ubuntu 16.04
* Git
<<<<<<< HEAD
* PHP 7.1
=======
* PHP 7.0
>>>>>>> a64be4f1e445bc790f1f0f2d692849d920956633
* Nginx
* MySQL
* MariaDB
* Sqlite3
* Postgres
* Composer
<<<<<<< HEAD
* Node (With Yarn, PM2, Bower, Grunt, and Gulp)
=======
* Node (With PM2, Bower, Grunt, and Gulp)
>>>>>>> a64be4f1e445bc790f1f0f2d692849d920956633
* Redis
* Memcached
* Beanstalkd

Over time you can modify this to your requirements and repackage the changes to your own, custom Vagrant box.

### Prerequisites
First you'll need VirtualBox and Vagrant. On Windows, I recommend installing these using the [Chocolatey](https://chocolatey.org) package manager. Why? We want everything to be easily repeatable so it can be done on multiple PCs.

#### Installing Chocolatey

1. Open a command prompt **as Administrator**
2. Type: `@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin`
3. Once Chocolatey has installed, run `chocolatey /?` to see if it has installed correctly and is on your PATH. If not, open a new command prompt as Administrator. This ensures the environment variables set by Chocolatey are loaded.

#### Installing Required Prerequisites
1. Again in an Administrator-level command prompt, type: `cinst virtualbox vagrant git.install consolez`

  This will take some time as the Vagrant and VirtualBox packages are quite big.

  **NOTE:** I will explore using Hyper-V instead of VirtualBox at some point. I've chosen VirtualBox for now as it has wider support. Hyper-V seems worth exploring due to alleged performance benefits. You also get PowerShell Cmdlets for Hyper-V which will make performing some VM automation tasks easier.

  **NOTE:** Some recent builds of VirtualBox have been problematic. If it is not working (i.e. VBoxGUI doesn't start or VM won't power on) you may need to uninstall it and manually reinstall an earlier version or a [test build](https://www.virtualbox.org/wiki/Testbuilds).

  Installation of git and ConsoleZ **are optional** - they are just my preferred options.

  If anything happens during the installation of any of the above packages, and you have to reinstall, just add -force to the end of the command to reinstall, e.g.

  `cinst virtualbox -force`

  If you don't want to download the packages multiple times, after the first PC, copy the following directory to each other PC in the same location:

  `%TEMP%\chocolatey\`

2. If you installed git, it does add its bin directory path to the machine-level PATH variable, however your command prompt won't register this immediately. The easiest thing to do is to reopen your Admin command prompt. You could also set the PATH like so:

  `set PATH=%PATH%;"c:\Program Files (x86)\Git\bin";"c:\Program Files (x86)\Git\cmd"`

#### Adding Vagrant Plugins

The following vagrant plugins will make some steps easier.

1. hostsupdater
2. cachier

hostsupdater automatically updates your Windows hosts file with network mappings found in Vagrant files.

cachier caches Vagrant boxes locally to save them being downloaded more than once.

Type: 

`vagrant plugin install vagrant-hostsupdater`

and

`vagrant plugin install cachier`

See [full list of Vagrant plugins](https://github.com/mitchellh/vagrant/wiki/Available-Vagrant-Plugins)

### Installing Homestead

1. Download the Homestead Vagrant box Using Vagrant

   Open a command prompt and type: `vagrant box add laravel/homestead`. 

   This box currently supports the virtualbox and vmware_desktop providers. Choose whichever you like, although the remaining instructions here only cover VirtualBox (for now.)

   You can use `vagrant box add laravel/homestead --provider virtualbox` as a shortcut.

   This will take some time to download. The box contains the base operating system and components, without your specific configuration.

   If your download is interrupted you can restart it with the above command, however some servers don't support resuming. In this case use `vagrant box add laravel/homestead --clean` to restart the download.

2. **Optional** Manual Download of Vagrant Box

   If this takes too long to download, you can download using a download manager, for example: [Free Download Manager](http://www.freedownloadmanager.org/) or [Aria2](https://chocolatey.org/packages/aria2). You can find the URL to download from the output of the `vagrant box add` command above (Ctrl-C to cancel the vagrant download):

   ![vagrant box add](http://i.gyazo.com/a02efb5e926c55e8d950eda815382d48.png "vagrant box add command")

   Use the download manager to download the box directly from `https://atlas.hashicorp.com/laravel/boxes/homestead/versions/0.5.0/providers/virtualbox.box`.

   You must then add it to the local vagrant box index. To do this:

   `vagrant box add c:\path\to\downloaded\box\virtualbox.box --name "laravel\homestead"`. The **name is very important** because it is used to load the box from your Laravel Homestead configuration (see below.)

   This will import the box and place its extracted contents in `%USERPROFILE\.vagrant.d\boxes\laravel-VAGRANTSLASH-homestead\[current_version]\virtualbox\`.

   Once you've downloaded **and added** the box once, you can copy it to other machines from the location: `%USERPROFILE\.vagrant.d\boxes\laravel-VAGRANTSLASH-homestead\`. Copy the entire directory to the same location on the other Windows PC's.

3. Next we download the Vagrant configuration from github.

   The recommended way is to keep this in a central location which will be used for all Laravel projects.

  Open a command prompt and navigate to your **dev_root** directory.

   Type: `git clone https://github.com/laravel/homestead.git homestead`. This command will clone the configuration project into a directory called homestead.

   ![Get vagrant project for homestead](http://i.gyazo.com/56142a7daa7a611d08c0ea636ca980a3.png)

   Now you have a directory called **dev_root**\homestead. The idea is that this is reused for all your Laravel projects. You'll see shortly how to configure mappings to each of your projects.

4. Generate the Homestead configuration file (YAML)

  The above project ships with a BASH script for initialising the project on your PC. Therefore this won't work on Windows. Luckily it's a very simple script, so we can easily replicate it in a command prompt:

  ```
  # in dev_root\homestead
  mkdir %userprofile%\.homestead
  copy src\stubs\Homestead.yaml %userprofile%\.homestead
  copy src\stubs\after.sh %userprofile%\.homestead
  copy src\stubs\aliases %userprofile%\.homestead
  ```

  or in PowerShell:

  ```
  # in dev_root\homestead
  mkdir $env:userprofile\.homestead
  copy src\stubs\Homestead.yaml $env:userprofile\.homestead
  copy src\stubs\after.sh $env:userprofile\.homestead
  copy src\stubs\aliases $env:userprofile\.homestead
  ```

  The idea is that you should never make local modifications to the laravel/homestead repo (unless you are contributing.) Therefore you need to store your local config in `%userprofile%\.homestead`.

5. Create your Laravel project storage directory, for example **dev_root**\laravel\

    `mkdir laravel`

6. Change the configuration files to match your environment:

    **Keys**

    Most people like to use their own keypairs, so if you don't have yours in the default location, you'll need to change the path accordingly for the "authorize" and "keys" configuration items.

    The defaults are:

    ```
    authorize: ~/.ssh/id_rsa.pub

    keys:
        - ~/.ssh/id_rsa
    ```

    On Windows ~ resolves as %USERPROFILE% so the above will work. Git and other ssh-requiring software also use this location so I'd recommend storing your keys there.

    You can generate a keypair by opening the Git Bash shell and typing:

    ```
    ssh-keygen -t rsa -C "MY_NAME@homestead"
    ```

    Substitute MY_NAME for your username. ssh-keygen will ask you where to save the file and the default option may not work. If not, just type in the full path, e.g. `c:\users\joe\.ssh\id_rsa`.
    
   **Folders**

   We need to change the folder sharing to use our own **dev_root**.

   Also, because we're on Windows, we need to make some changes to the path format.

   **Note:** the Windows path format in a yaml file is like this: `C:/Users/me/dev`

   1. Open `%userprofile%/Homestead.yaml` in a text editor
   2. Change "map:" under "folders:" to **dev_root**/laravel
      
      This path is case sensitive. Note we've used Linux-style forward slashes for the path, e.g. `c:/Users/joe/dev`

   3. Change "to:" under "folders:" to "/home/vagrant/projects"

   `/home/vagrant/projects` is the location of our web projects inside the Homestead Linux VM.

   What we are doing here is exposing the contents of the Windows **dev_root**\laravel\ directory to the VM and making it available at _/home/vagrant/projects_

   You can have as many of these folder map/to pairs as you want, but because we've mapped the project root you may only need one.

   **Sites**

   The sites section tells the Homestead VM's nginx service which sites to serve within our VM, by defining the mapping between a local domain and a folder. You can add as many sites as you want.

   1. Change "map:" and "to:" under "sites:" to map laravel-api.phplocal.dev to your Laravel project root for laravel-api, e.g.:

   ```
   map: laravel-api.phplocal.dev
   to: /home/vagrant/projects/laravel-api/public
   ```

   This section is used during Vagrant provisioning (within `scripts/serve.sh`) to create the nginx site configuration. You can define multiple sites if you need to.

   **Note:** the above directory doesn't exist yet! We'll create it in the next document [Working with Laravel](Working-With-Laravel.md).

   **Other**

   1. You may also like to change the memory value to 1024 (or less) if your host machine is light on memory.
   
   2. You can optionally add additional databases, just by adding to the list under `databases:`.

    If you want to create additional users for database access, this is done by adding the required commands to the `after.sh` file in `%userprofile%\.homestead\`:

    ```
    mysql -uhomestead -psecret db2 -e "
    GRANT ALL PRIVILEGES ON db2.* TO db2_usr@localhost IDENTIFIED BY 'akjshdf7767df';
    FLUSH PRIVILEGES;"
    ```

   3. To see the full list of settings, check out **scripts\homestead.rb**. Most of the settings in here can be overwritten by values in the yaml file, but the default port mappings are hard coded.

    **Remember for later:** your website will be accessed on port 8000.

   ##### Example Homestead.yaml:

    ```
    ---
    ip: "192.168.10.10"
    memory: 2048
    cpus: 1
    provider: virtualbox

    authorize: ~/.ssh/id_rsa.pub

    keys:
        - ~/.ssh/id_rsa

    folders:
        - map: C:/Users/Joe/dev/laravel
          to: /home/vagrant/projects

    sites:
        - map: laravel-api.phplocal.dev
          to: /home/vagrant/projects/laravel-api/public

    databases:
        - homestead

    variables:
        - key: APP_ENV
          value: local

    # blackfire:
    #     - id: foo
    #       token: bar
    #       client-id: foo
    #       client-token: bar

    # ports:
    #     - send: 93000
    #       to: 9300
    #     - send: 7777
    #       to: 777
    #       protocol: udp
    ```

5. Add the above host name to your hosts file:

  File: c:\windows\system32\drivers\etc\hosts

  ```
  192.168.10.10 laravel-api.phplocal.dev
  ```

6. **Optional:** Configure blackfire profiler:
    
    If you want to use blackfire, you need to sign up at https://blackfire.io/signup. Go through the process then go to https://blackfire.io/account/credentials and copy the relevant info to `homestead.yaml`.

    You also will probably want to install the [Chrome extensions](https://chrome.google.com/webstore/detail/blackfire-companion/miefikpgahefdbcgoiicnmpbeeomffld)    

7. **Optional:** if you don't want to bother using your own SSH key, you can comment out the key deployment part in the `homestead.rb` file found in the scripts/ directory. If you do this, vagrant will detect the insecure key during provisioning and replace it with a new key pair. `vagrant ssh` will automatically use this pair. This should be an OK and easier option if you are a solo developer.

  Comment out these lines (from line 72 in the [current version](https://github.com/laravel/homestead/blob/351aebd8bdaf5bb87d76018228b3427b56a7434d/scripts/homestead.rb#L72) of this file):

  ```
    # Configure The Public Key For SSH Access
    if settings.include? 'authorize'
      config.vm.provision "shell" do |s|
        s.inline = "echo $1 | grep -xq \"$1\" /home/vagrant/.ssh/authorized_keys || echo $1 | tee -a /home/vagrant/.ssh/authorized_keys"
        s.args = [File.read(File.expand_path(settings["authorize"]))]
      end
    end

    # Copy The SSH Private Keys To The Box
    if settings.include? 'keys'
      settings["keys"].each do |key|
        config.vm.provision "shell" do |s|
          s.privileged = false
          s.inline = "echo \"$1\" > /home/vagrant/.ssh/$2 && chmod 600 /home/vagrant/.ssh/$2"
          s.args = [File.read(File.expand_path(key)), key.split('/').last]
        end
      end
    end
  ```

7. **Now we're ready to start up the VM and get to work!** 

  Simply navigate to `**dev_root**\homestead\` and type `vagrant up`.

**Continue to [Working with Laravel](Working-With-Laravel.md) for the next steps.**

Troubleshooting
----------------

**Your synced folders are not available in the vagrant VM**

Solution: exit vagrant ssh and run `vagrant reload` try again.

If this doesn't work, double check the paths you have specified in your Homestead.yaml file.

Updating the Vagrant Box
--------------------------
From time to time, Laravel will release updates to the homestead box. These can be downloaded using the approach above or by running `vagrant box update` from your homestead directory.

### Updating via Vagrant (preferred)

If you want to update the box via `vagrant box update` you'll need to replace the box you're using with the versioned one. This can be done with the following command: `vagrant box add --force "laravel/homestead"`

### Updating by downloading the box manually

If you have a slow connection you may like to this method.

1. Download the box manually using aria2, Free Download Manager, etc., using the instructions above.
2. Once you have the file virtualbox.box, type the following: `vagrant box add laravel-VAGRANTSLASH-homestead c:\temp\virtualbox.box --force`

  It seems `vagrant box add laravel/homestead c:\temp\virtualbox.box --force` should work but I had to use `-VAGRANTSLASH-` instead with Vagrant version 1.7.3.
3. This will add the Vagrant box to: `%USERPROFILE\.vagrant.d\boxes\laravel-VAGRANTSLASH-homestead\0\virtualbox\`

  This is because adding the box in this way doesn't support specification of a box version. The next steps shows how to add this version.

4. Now copy `%USERPROFILE\.vagrant.d\boxes\laravel-VAGRANTSLASH-homestead\0\virtualbox\` to `%USERPROFILE\.vagrant.d\boxes\laravel-VAGRANTSLASH-homestead\[current_version]\virtualbox\`

  For example, you might copy it to: `%USERPROFILE\.vagrant.d\boxes\laravel-VAGRANTSLASH-homestead\0.3.0\virtualbox\`

Once you have the new box, you must replace your existing homestead instance. This is safe because the only unique data that the homestead contains is our databases. Our web project data is stored on our host machine. These can be repopulated using our seed scripts, so it's safe to destroy the VM and reprovision it. To do this:

```
vagrant reload
vagrant ssh

# from within the ssh session:
php artisan migrate
php artisan db:seed
```

Now we are right back where we were before updating the box!

Updating the Homestead configuration repo
--------------------------------------------

Simple:

`git pull`

If you have made modifications, save them elsewhere if you need them, and then:

```
git fetch origin
git reset --hard origin/master
```

Remember to copy the local config files to `%userprofile%\.homestead` and merge with your existing config.

2. (Optional) Native Windows Installation
-------------------------------------------
This section explains how to install Laravel directly on your machine. Method 1 is the recommended approach but this may be useful in some cases.

### Installing PHP

First you will need PHP installed on your development machine.

You can use Chocolatey to install PHP 5.6:

`cinst php -version 5.6.17`

  ![PHP 5.6 via Chocolatey](http://i.gyazo.com/3d858da9d02b8c5373f051454a38d1bb.png)

or PHP 7.0.x:

`cinst php`


### Installing Laravel

1. Install Composer from https://getcomposer.org/Composer-Setup.exe

  Or Chocolatey: `cinst composer`

  If you only have one PHP installation on the PC, the Chocolatey package should work fine.

2. Composer will ask for your php.exe location. If you only have one php.exe in your PATH, it should correctly autodetect the above PHP 5.6 installation.

  If you used the Microsoft installer above, the path to php.exe is `c:\program files (x86)\iis express\php\php.exe`

3. The composer installer will add itself to the PATH. Open a new command prompt to load the updated PATH, or if you need to add it yourself the path to add is: `c:\programdata\ComposerSetup\bin`

4. Type `composer global require "laravel/installer=~1.1`. This will install laravel into %USERPROFILE%\.composer\vendor\bin\

  ![Downloading Laravel via Composer](http://i.gyazo.com/8443d590de5fe190df40bff729151482.png)

5. Laravel is now installed and ready for project creation.

**Continue to [Working with Laravel](Working-With-Laravel.md) for the next steps. These are currently only for Homestead but the steps are similar when working natively in Windows.**
