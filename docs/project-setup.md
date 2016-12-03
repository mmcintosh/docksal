# Configure a project to use Docksal

With Docksal you can initialize a basic LAMP stack with no configuration.   
In this case the default configuration will be used to provision containers and set up a virtual host.

Initial configuration is done once per project (e.g. by a team lead) and committed into the project repo. 
Presence of the `.docksal` folder in the project directory is a good indicator the project is using Docksal.

## Default setup

1) Create a project directory

```
mkdir ~/projects/myproject  
cd ~/projects/myproject
```  

2) Create an empty `.docksal` directory

```
mkdir .docksal
```

All project-specific configurations and commands will be stored in this directory.

**Note:** Git does not commit empty directories.
To commit `.docksal` directory to git create a `.gitkeep` file inside it:  

```
touch .docksal/.gitkeep
```

3) Start the project containers

```
fin start
```

You should see output like the following:

```
Starting services...
Creating network "myproject_default" with the default driver
Creating volume "myproject_project_root" with local driver
Creating volume "myproject_host_home" with local driver
Creating myproject_cli_1
Creating myproject_db_1
Creating myproject_web_1
Changing user id in cli to 501 to match host user id...
Resetting permissions on /var/www...
Restarting php daemon...
Connected vhost-proxy to "myproject_default" network.
```

Your project site is now running. If you visit the project url `http://myproject.docksal` you will get a 404 error, because nothing is there yet!

**Note: SSH keys passphrase** 
If you are asked for the SSH keys passphrase for `id_dsa` or `id_rsa`, 
please know that these are **your** keys, that were copied over from your `~/.ssh` folder 
into the `ssh-agent` container. That's why their paths looks like `/root/.ssh/...`, 
because that's the path **inside the container**. 
Please provide password(s) if you want to use git or drush commands, that require ssh access within Docksal 
(e.g. often project init script or composer script contains repository checkout, 
that would require an ssh key for access).

4) Setup document root

Add a document root and start adding files.

`mkdir docroot`

Add any project files you want to `docroot`: plain HTML, PHP-based CMS or pure PHP project.


## Checking the default configuration

If you setup a project using the simplified process above, Docksal handles all the configuration
behind the scenes. To review the configuration, type `fin config` in your project directory.

You will see an output similar to the following:

```
COMPOSE_PROJECT_NAME: myproject
COMPOSE_PROJECT_NAME_SAFE: myproject
COMPOSE_FILE:
/Users/testuser/.docksal/stacks/volumes-bind.yml
/Users/testuser/.docksal/stacks/stack-default.yml
ENV_FILE:


PROJECT_ROOT: /Users/testuser/projects/myproject
DOCROOT: docroot
VIRTUAL_HOST: myproject.docksal
IP: 192.168.64.100

MYSQL_PORT:

Docker Compose configuration
---------------------
networks: {}
services:
  cli:
    hostname: cli
    image: docksal/cli:1.0-php7
    volumes:
    - host_home:/.home:ro
    - docksal_ssh_agent:/.ssh-agent:ro
    - project_root:/var/www:rw
  db:
    environment:
      MYSQL_DATABASE: default
      MYSQL_PASSWORD: user
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: user
    hostname: db
    image: docksal/db:1.0-mysql-5.5
  web:
    depends_on:
    - cli
    environment:
      APACHE_DOCUMENTROOT: /var/www/docroot
      VIRTUAL_HOST: myproject.docksal
    hostname: web
    image: docksal/web:1.0-apache2.2
    labels:
      io.docksal.project-root: /Users/testuser/projects/myproject
      io.docksal.virtual-host: myproject.docksal
    volumes:
    - project_root:/var/www:ro
version: '2.0'
volumes:
  docksal_ssh_agent:
    external: true
    external_name: docksal_ssh_agent
  host_home:
    driver: local
    driver_opts:
      device: /Users/testuser
      o: bind
      type: none
  project_root:
    driver: local
    driver_opts:
      device: /Users/testuser/projects/myproject
      o: bind
      type: none

---------------------
```

Notice it displays the virtual host name it will use, which is based on your
project's directory name, and it displays the MySQL user name and password if
you need to configure database access.

It also displays the configuration for the default LAMP stack containers: cli, db, and web.


## Customizing the stack configuration

To customize the project's Docksal stack, you first have generate the config files:

```
fin config generate
```

This will generate two files in the `.docksal` directory inside the project:

- `docksal.env` - used for environment specific configuration, like setting the document root
or hostname.
- `docksal.yml` - used for Docker specific configuration, like adding or removing services.


## Automate the initialization process

This is optional, but highly recommended.

Site provisioning can be automated via a [custom command](custom-commands.md).  
E.g. `fin init`, which will call `.docksal/commands/init`.

Put project specific initialization tasks there, like:

- initialize Docksal configuration
- import database or perform a site install
- compile SASS
- run DB updates, special commands, etc.
- run Behat tests

For a fully working example of a Docksal powered project (including `fin init`) take a look at:

- [Drupal 7 sample project](https://github.com/docksal/drupal7)
- [Drupal 8 sample project](https://github.com/docksal/drupal8)
- [WordPress sample project](https://github.com/docksal/wordpress)
