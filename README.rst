.. contents:: Table of contents

Overview
========

Current `OXID eShop <http://www.oxid-esales.com/en/home.html>`_ development
environment is inspired by `PuPHPet <https://puphpet.com/>`_ and
`Phansible <http://phansible.com/>`_ projects.

Virtual environment is built using:

* `Vagrant <https://www.vagrantup.com/>`_ - virtual environment automation tool;
* `Ansible <http://www.ansible.com/>`_ - environment orchestration tool;
* `YAML <http://yaml.org/>`_ - solution configuration.

Final solution is composed of two repositories (*linked using git sub-modules*):

* `Base VM <https://github.com/OXID-eSales/oxvm_base>`_ - Base LAMP stack
  (*also used as base for other VMs*);
* `eShop VM <https://github.com/OXID-eSales/oxvm_eshop>`_ - Current repository,
  eShop specific configuration, roles and
  `SDK components <http://wiki.oxidforge.org/SDK>`_.

Getting started
===============

Be sure to install the following **minimal dependencies**:

* `Vagrant <https://www.vagrantup.com/downloads.html>`_ (>=1.7)
* `VirtualBox <https://www.virtualbox.org/>`_ [#virtualbox_dependency]_ (>=4.2)
* `Git <https://git-scm.com/downloads>`_
* `OpenSSH <http://www.openssh.com/>`_

Quick start
-----------

* Install necessary **vagrant plugins**:

.. code:: bash

  vagrant plugin install vagrant-hostmanager
  vagrant plugin install vagrant-bindfs
  vagrant plugin install vagrant-triggers

* Clone [#recursive_clone]_ out current repository:

.. code:: bash

  git clone --recursive https://github.com/OXID-eSales/oxvm_eshop.git

* Start the VM:

.. code:: bash

  cd oxvm_eshop
  vagrant up

* After successful provision process use the following links to:

  * Open OXID eShop: http://www.oxideshop.dev/
  * Access admin area: http://www.oxideshop.dev/admin/

    * Username: ``admin``
    * Password: ``admin``

.. [#virtualbox_dependency] VirtualBox is listed as dependency due to the fact
  that it is the default chosen provider for the VM. In case other providers
  will be used there is no need to install VirtualBox. Please refer to the list
  of possible providers in the configuration section to get more information.
.. [#recursive_clone] Since the current eShop VM repository is linked through git sub-modules
  it is mandatory to use ``--recursive`` option to instruct ``git`` and clone
  base VM repository as well.

Configuration
=============

Default virtual environment configuration should be sufficient enough to get
the eShop CE/PE/EE versions up and running. However, it is possible to adjust
the configuration of virtual environment to better match personal preferences.

All configuration changes should be done by overriding variables from:

* `default.yml <https://github.com/OXID-eSales/oxvm_base/blob/master/ansible/vars/default.yml>`_ - base vm variables;
* `oxideshop.yml <https://github.com/OXID-eSales/oxvm_eshop/blob/master/ansible/vars/oxideshop.yml>`_ - eShop specific variables.

These overridden values must be placed in ``personal.yml``
[#personal_git_ignore]_ file at the root level of current repository.

For the overridden values to take effect please run ``vagrant provision``. If
the changes are related to the shared folder use ``vagrant reload``. In case the
provision process will start to show any kind of errors, please try to use
``vagrant destroy && vagrant up`` for the process to stat over from a clean
state.

Examples
--------

Below is a list of possible frequent changes which are typically done after
cloning this repository.

One can just copy and paste the example snippets from the list below to the
``personal.yml`` file at the root of this repository.

Change VM provider
^^^^^^^^^^^^^^^^^^

Change VM provider from VirtualBox (*default*) to LXC.
A list of available and tested providers [#list_of_providers]_:

- `virtualbox <https://www.virtualbox.org/>`_ - Default provider which is free
  to use and available on all major operating systems;
- `lxc <https://linuxcontainers.org/>`_ [#lxc_provider]_ - Operating system
  level virtualization which vastly improves I/O performance compared to
  para-virtualization solutions;
- `parallels <http://www.parallels.com/eu/>`_ [#parallels_provider]_ - Commercial
  VM provider for OS X.

.. code:: yaml

  ---
  vagrant_local:
    vm:
      provider: lxc

Set eShop to UTF-8 mode
^^^^^^^^^^^^^^^^^^^^^^^

By default shop will be installed with UTF-8 mode disabled (*with*
``iUtfMode = '0'`` *value inside* ``config.inc.php``).

In order to turn on the UTF-8 mode:

.. code:: yaml

  ---
  eshop:
    config:
      utf_mode: 1

This change will not affect the already configured shop [#turn_on_utf_mode]_ .

Change shared folder path
^^^^^^^^^^^^^^^^^^^^^^^^^

Change the default application shared folder from ``oxideshop`` to local path
``/var/www`` and update eShop target folder [#eshop_target]_.

.. code:: yaml

  ---
  vagrant_local:
    vm:
      app_shared_folder:
        source: /var/www
        target: /var/www
  eshop_target_path: /var/www/oxideshop

Define github token for composer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Provide OAuth token from github for composer so that the access API limit could
be removed [#github_token]_.

.. code:: yaml

  ---
  php:
    composer:
      github_token: example_secret_token

Change ubuntu repository mirror url
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Change the default ubuntu repository mirror url from ``http://us.archive.ubuntu.com/ubuntu/``
to ``http://de.archive.ubuntu.com/ubuntu/``.

.. code:: yaml

  ---
  server:
    apt_mirror: http://de.archive.ubuntu.com/ubuntu/

Change virtual host
^^^^^^^^^^^^^^^^^^^

Change the default virtual host from ``www.oxideshop.dev`` to
``www.myproject.dev``.

.. code:: yaml

  ---
  vagrant_local:
    vm:
      aliases:
        - www.myproject.dev

Change MySQL password
^^^^^^^^^^^^^^^^^^^^^

Change the default MySQL user password from ``oxid`` to ``secret``.

.. code:: yaml

  ---
  mysql:
    password: secret

Trigger Varnish installation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Trigger `Varnish <https://www.varnish-cache.org/>`_ [#varnish_usage]_
installation so that it can be used within eShop.

.. code:: yaml

  ---
  varnish:
    install: true

Trigger Selenium installation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Trigger `Selenium <http://www.seleniumhq.org/>`_ installation so that it can be
used to run Selenium tests with the help of
`OXID testing library <https://github.com/OXID-eSales/testing_library.git>`_.

.. code:: yaml

  ---
  selenium:
    install: true

.. [#personal_git_ignore] ``personal.yml`` configuration file is already
  included in ``.gitignore`` and should not be visible as changes to the actual
  repository.
.. [#list_of_providers] VM solutions from `VMWare <http://www.vmware.com/>`_,
  such as `workstation <http://www.vmware.com/products/workstation>`_ and
  `fusion <http://www.vmware.com/products/fusion>`_ were not yet adapted or
  tested with our current configuration of VM.
.. [#lxc_provider] Keep in mind that LXC provider is only available for
  GNU/Linux based operating systems. In order to start using this provider with
  vagrant a plugin must be installed for it
  (``vagrant plugin install vagrant-lxc``). So far it has been only tested with
  Ubuntu based OS with lxc package installed (``sudo apt-get install lxc``).
.. [#parallels_provider] A vagrant plugin must be installed
  (``vagrant plugin install vagrant-parallels``) in order to use vagrant with
  Parallels.
.. [#turn_on_utf_mode] Keep in mind that the provided snippet will not change
  the UTF-8 mode of the eShop if the configuration file (``config.inc.php``) is
  already present and defined. In this case one has to make the change of
  ``iUtfMode`` value directly in the ``config.inc.php`` file of the eShop.
.. [#eshop_target] Keep in mind that if the shared folder target does not match
  actual application (eShop) target it has to be specified explicitly by
  defining ``eshop_target_path``.
.. [#github_token] By default github has API access limits set for anonymous
  access. In order to overcome these limits one has to create a github token,
  which could be done as described in:
  https://help.github.com/articles/creating-an-access-token-for-command-line-use/
.. [#varnish_usage] Varnish can only be used with the eShop EE version and with
  purchased "performance pack" (https://www.oxid-esales.com/performance/). Keep
  in mind that the default Varnish port 6081 is being used to access the shop.
  This should also be reflected in ``config.inc.php`` file as ``sShopURL``
  parameter, e.g. http://www.oxideshop.dev:6081/ .

SDK
===

Out of the box the VM is equipped with the following SDK components:

* `Module skeleton generator <https://github.com/OXID-eSales/module_skeleton_generator>`_ - module
  which helps to create new OXID eShop modules;
* `Module certification tools <https://github.com/OXID-eSales/module_certification_tools>`_ - a
  collection of tools which allows one to see a detailed report from module
  certification process;
* `Testing library <https://github.com/OXID-eSales/testing_library>`_ - a
  library for writing various kind of tests inside eShop and a set of tools for
  running those tests.

There are also other SDK components which could be found at:
http://wiki.oxidforge.org/SDK

Usage
-----

Module skeleton generator
^^^^^^^^^^^^^^^^^^^^^^^^^

By default this module is installed under eShop's ``modules`` directory (by
default it will be ``/var/www/oxideshop/source/modules/`` which is defined by
``eshop_path.modules`` key in configuration).

The module needs to be activated manually. Further instructions on how to enable
and use the module could be found at (*installation part can be skipped*):
https://github.com/OXID-eSales/module_skeleton_generator#usage

Module certification tools
^^^^^^^^^^^^^^^^^^^^^^^^^^

By default the tools are installed under VM's home folder (``~/eshop_sdk`` which
is defined by ``eshop.sdk.path`` key in configuration). The repository of tools
is cloned out in ``~/eshop_sdk/module_certification_tools`` and an extra
shortcut ``ox_cert`` is created inside ``~/eshop_sdk/bin/`` (it's included in
``PATH`` environment variable automatically).

There is no need to do any installation part for tools to work as it is already
done by the VM's provision process.

In order to invoke the certification report generator just use the provided
shortcut:

``ox_cert <vendor_name>/<module_name>``

An example of invoking the reporting tool for module generator
[#cert_tools_call]_:

.. code:: bash

  $ ox_cert oxps/modulegenerator

After the execution it will generate a HTML document which will be placed at
``~/eshop_sdk/module_certification_tools/result/<datetime>/report.html``.

Once the report is generated one can just view the contents of it straight
from inside the VM using command line tools or copy the file to shared folder
and view it from host machine, e.g.:

.. code:: bash

  cp ~/eshop_sdk/module_certification_tools/result/20150916101719/report.html \
    /var/www/oxideshop

Testing library
^^^^^^^^^^^^^^^

Library needed for various testing purposes is already installed in the VM
through the help of `composer <https://getcomposer.org/>`_, because it's defined
in ``composer.json`` as development requirement inside eShop (at least in CE
latest development version).

All binary tools are installed inside ``/var/www/oxideshop/source/vendor/bin/``
(this value may be changed through ``eshop_path.source`` key in configuration).

A list of available binary tools:

* ``reset-shop`` - restore eShop's database to it's default state (demo);
* ``runmetrics`` - run `pdepend <http://pdepend.org/>`_ against eShop and
  modules code to collect various code metrics information;
* ``runtests`` - run unit/integartion tests against eShop and modules code;
* ``runtests-coverage`` - generate coverage report by running unit/integration
  tests;
* ``runtests-selenium`` - run acceptance tests written for Selenium.

More details on how to use and configure the library could be found at:
https://github.com/OXID-eSales/testing_library

.. [#cert_tools_call] The tools can be invoked from any working directory as
  long as the ``ox_cert`` shortcut is being used.

Guides
======

List of guides for working with VM:

Use eShop packages instead of repositories
------------------------------------------

Our current `eShop packages <http://wiki.oxidforge.org/Downloads/4.9.5_5.2.5>`_
have different file/directory structure compared to
`eShop repositories <https://github.com/OXID-eSales/oxideshop_ce>`_. VM is
suited to work for eShop repository file/directory structure (*because it
holds our source and test files at the same place*). Due to this reason one
would need to manually extract eShop source and test packages into shared
folder.

In order to prepare VM for usage of eShop packages please follow the steps below
**before** running the VM:

* Create shared folder [#shared_folder]_ (*By default it's* ``oxideshop``
  *folder*);
* Extract source package into ``oxideshop/source`` folder;
* Extract tests package into ``oxideshop/tests`` folder
  (*This step is optional for eShop runtime*).

If the above steps were done **after** the creation/provision of VM please make
sure to re-run ``vagrant provision`` command for provision process to make
necessary changes.

**Directory structure overview of eShop packages placed for the VM**:

.. code:: bash

  oxvm_eshop - root of oxvm_eshop repository
  + <oxvm_eshop repository files>
  + oxideshop - shared folder
     + source
        + <eShop source package contents>
     + tests
        + <eShop tests package contents>

**An example of commands to prepare VM for using eShop packages**:

Keep in mind that the below example only demonstrates how one should prepare
the VM for source and test packages. In order to actually get/download source
and test packages for eShop PE/EE versions please contact
`OXID eSales support <https://www.oxid-esales.com/en/support-services/software-maintenance-and-support.html>`_.

In case the following two packages were received:

* ``OXID_ESHOP_EE_5.2.5_for_PHP_5.6.zip``  - source package
* ``OXID_ESHOP_TESTS_EE_5.2.5_for_PHP_5.6_SOURCE.rar`` - test package

.. code:: bash

  # Clone out VM repository
  git clone --recursive https://github.com/OXID-eSales/oxvm_eshop.git
  cd oxvm_eshop

  # Download packages
  wget http://<url provided by support>/OXID_ESHOP_EE_5.2.5_for_PHP_5.6.zip
  wget http://<url provided by support>/OXID_ESHOP_TESTS_EE_5.2.5_for_PHP_5.6_SOURCE.rar

  # Extract packages
  mkdir oxideshop
  unzip OXID_ESHOP_TESTS_EE_5.2.5_for_PHP_5.6_SOURCE.rar -d ./oxideshop/source/
  unrar x OXID_ESHOP_TESTS_EE_5.2.5_for_PHP_5.6_SOURCE.rar ./oxideshop/tests/

  # Start the VM
  vagrant up

.. [#shared_folder] The actual sharing process of the folder will be done by
  the VM provision process, end-user only needs to create this folder and make
  sure the folder exists. The path and name of the folder is configurable via
  the ``vagrant_local.vm.app_shared_folder`` parameter. By default it's
  ``<oxvm_eshop_root>/oxideshop``. More information about how to configure this
  value can be found in:
  `Change shared folder path <#change-shared-folder-path>`_ .

Troubleshooting
===============

List of troubleshooting items:

Provision process hangs on "Run composer install" task
------------------------------------------------------

During the provision process (*which could be invoked implicitly by*
``vagrant up`` *or explicitly by* ``vagrant provision``) a task ``Run composer
install`` might hang (*waiting for time-out*) because github access API limit
has been reached and ``composer`` is asking for github account username/password
which could resolve the API limit. ``Ansible`` will not provide this information
to ``STDOUT`` or ``STDERR`` so it will look like the task just hanged.

Since there are no options to provide username/password for this particular task
one could just use a github API token which will allow to overcome the API
access limit.

How to create and configure a github token is described in
`Define github token for composer <#define-github-token-for-composer>`_ chapter.

Error from `Unit_Admin_ModuleListTest::testRender()` method while testing eShop
-------------------------------------------------------------------------------

Older versions of eShop contains a very strict test inside
`Unit_Admin_ModuleListTest::testRender()` method which tries to match the exact
list of available modules. The test method might fail because VM includes SDK
components and some of them are actual modules (*which will result in modified
list of available modules*).

This is a known issue which is fixed in the development and new upcoming
releases of eShop.

To check which shop is compatible with testing library please refer to `compatibility list <https://github.com/OXID-eSales/testing_library/tree/b-1.0#compatibility-with-oxid-shops>`_.

Browser shows "Zend Guard Run-time support missing!"
----------------------------------------------------

This message will only appear if a
`Zend Guard <https://www.zend.com/en/products/zend-guard>`_ encoded eShop
package is being used. In order to solve the issue one has to install
`Zend Guard Loader <http://www.zend.com/en/products/loader/downloads>`_ which
will decode the encoded PHP files on execution.

To install and enable Zend Guard Loader PHP extension inside VM:

.. code:: bash

  # From host (local machine)
  vagrant ssh

  # From guest (virtual machine)
  cd /usr/lib/php5/20121212/
  sudo wget https://github.com/OXID-eSales/oxvm_assets/blob/master/zend-loader-php5.5-linux-x86_64.tar.gz?raw=true -O zend.tar.gz
  sudo tar zxvf zend.tar.gz
  sudo cp zend-loader-php5.5-linux-x86_64/ZendGuardLoader.so ./
  sudo cp zend-loader-php5.5-linux-x86_64/opcache.so ./zend_opcache.so
  cd /etc/php5/mods-available/
  sudo sh -c 'echo "zend_extension=ZendGuardLoader.so" > zend.ini'
  sudo sh -c 'echo "zend_extension=zend_opcache.so" >> zend.ini'
  sudo php5dismod opcache
  sudo php5enmod zend
  sudo service apache2 restart

Keep in mind that different PHP version needs different version of Zend Guard
Loader extension. List of possible extension versions can be found in
`oxvm_assets <https://github.com/OXID-eSales/oxvm_assets>`_ repository.

More information on how to install and configure Zend Guard Loader can be found
at: http://files.zend.com/help/Zend-Guard/content/installing_zend_guard_loader.htm
