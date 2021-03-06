Setup System Requirements on Ubuntu 14.04
=========================================

Here is a quick guide to setup the :doc:`system_requirements` on Ubuntu 14.04.

System installation
-------------------

Base dependencies
*****************

.. code-block:: bash
    :linenos:

    $ sudo apt-get update
    $ sudo apt-get install mysql-server
    $ sudo apt-get install apache2
    $ sudo apt-get install libapache2-mod-php5 php5-cli
    $ sudo php5enmod mcrypt
    $ sudo a2enmod rewrite
    $ sudo apt-get install php5-apcu

.. note::
    PHP 5.5 provided in Ubuntu 14.04 comes with the Zend OPcache opcode cache. Only the data cache provided by APCu is needed.

As PHP 5.5 is provided in Ubuntu 14.04, you will have to upgrade to PHP 5.6 using:

.. code-block:: bash
    :linenos:

        $ sudo add-apt-repository ppa:ondrej/php
        $ sudo apt-get update
        $ sudo apt-get install php5.6
        $ sudo apt-get install php5.6-xml php5.6-zip php5.6-curl php5.6-mongo php5.6-intl php5.6-mbstring php5.6-mysql php5.6-gd php5.6-mcrypt

Check that PHP 5.6 is now your current PHP version with:

.. code-block:: bash
    :linenos:

        $ php -v

Choosing the product storage
****************************

.. include:: /reference/technical_information/choose_database.rst.inc

Based on this formula, either you need :ref:`installing-mongodb`, either you can directly go to the :ref:`system-configuration` section.

.. _installing-mongodb:

Installing MongoDB
******************

* Install MongoDB server

.. code-block:: bash
    :linenos:

    $ sudo apt-get update
    $ sudo apt-get install mongodb

.. note::

    Akeneo PIM will not work with MongoDB 3.*. *The supported versions are 2.4 and 2.6*.

* Install MongoDB PHP driver

.. code-block:: bash
    :linenos:

    $ sudo apt-get install php5-mongo


.. _system-configuration:

System configuration
--------------------

MySQL
*****

* Create a MySQL database and a user for the application

.. code-block:: bash
    :linenos:

    $ mysql -u root -p
    mysql> CREATE DATABASE akeneo_pim;
    mysql> GRANT ALL PRIVILEGES ON akeneo_pim.* TO akeneo_pim@localhost IDENTIFIED BY 'akeneo_pim';
    mysql> EXIT

PHP
***

.. include:: /reference/technical_information/php_ini.rst.inc

Apache
******

Setting-up the permissions
^^^^^^^^^^^^^^^^^^^^^^^^^^

To avoid spending too much time on permission problems between the CLI user and the Apache user, an easy configuration
is to use the same user for both processes.

* Get your identifiers

.. code-block:: bash
    :linenos:

    $ id
    uid=1000(my_user), gid=1000(my_group), ...

In this example, the user is *my_user* and the group is *my_group*.

* Stop Apache

.. code-block:: bash
    :linenos:

    $ sudo service apache2 stop

* Use your identifiers for Apache by editing the file ``/etc/apache2/envvars``. You have to replace the variables:

 * ``APACHE_RUN_USER=www-data`` by ``APACHE_RUN_USER=my_user``
 * ``APACHE_RUN_GROUP=www-data`` by ``APACHE_RUN_GROUP=my_group``

.. code-block:: bash
    :linenos:

    $ sudo gedit /etc/apache2/envvars
    # replace the environment variables
    export APACHE_RUN_USER=my_user
    export APACHE_RUN_GROUP=my_group
    $ sudo chown -R my_user /var/lock/apache2

* Restart Apache

.. code-block:: bash
    :linenos:

    $ sudo service apache2 start

Creating the virtual host file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create the file ``/etc/apache2/sites-available/akeneo-pim.local.conf``

.. code-block:: apache
    :linenos:

    <VirtualHost *:80>
        ServerName akeneo-pim.local

        DocumentRoot /path/to/installation/pim-community-standard/web/
        <Directory /path/to/installation/pim-community-standard/web/>
            AllowOverride All
            Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/akeneo-pim_error.log

        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/akeneo-pim_access.log combined
    </VirtualHost>

.. note::
    Replace */path/to/installation* by the path to the directory where you want to install the PIM.

.. note::
    Replace *pim-community-standard* by *pim-enterprise-standard* for enterprise edition.

.. note::
    Don't forget to add the ``web`` directory of your Symfony application.

Enabling the virtual host
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash
    :linenos:

    $ cd /etc/apache2/sites-available/
    $ sudo apache2ctl configtest
    $ sudo a2ensite akeneo-pim.local
    $ sudo service apache2 reload


Adding the virtual host name
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Edit the file ``/etc/hosts`` and add the following line

.. code-block:: bash
    :linenos:

    127.0.0.1    akeneo-pim.local
