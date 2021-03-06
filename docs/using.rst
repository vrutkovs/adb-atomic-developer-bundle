=================================
Using the Atomic Developer Bundle
=================================

Principles
==========

* Consider the ADB to be ephemeral and do not store data in it long term. It is
  recommended that you do the following:

  * back up your code
  * use a source control system
  * mount your code into the box from your host system

  Doing these things is beyond the scope of this document. Consult your
  Operating System manuals and the `Vagrant <http://vagrantup.com/>`_ website
  for more details.

Starting the Vagrant Box
========================

1. Initialize a new Vagrant environment by creating a Vagrantfile. You may find
   it helpful to first create a new directory for use by this environment.
   Create the Vagrantfile with this command:

   ``vagrant init <box name>``

   Box name is typically ``projectatomic/adb`` or whatever you named it when you
   loaded it manually.

2. Edit the ``Vagrantfile`` to setup private networking. The private network is
   used to expose ADB-based services, such as the docker daemon, to the host.
   This is done by adding the following line in the ``Vagrant.configure``
   section.

   ``config.vm.network "private_network", type: "dhcp"``

3. Start the Vagrant image with this command:

   ``vagrant up``

   **Note:** On Fedora and CentOS you may need to specify which virtualization
   provider to use.  For example, to use VirtualBox, the command would be
   ``vagrant up --provider virtualbox``

Using Custom Vagrantfiles for Specific Use Cases
================================================

Custom Vagrant files are shipped for specific use cases. The README files
contain useful documentation and a quickstart guide.

* Docker for use with host-based tools, such as Eclipse and the docker CLI, or
  via ``vagrant ssh``

  * `Vagrantfile <../components/centos/centos-docker-base-setup/Vagrantfile>`_
  * `README <../components/centos/centos-docker-base-setup/README.rst>`_

* Docker and Kubernetes for use with host-based tools or via ``vagrant ssh``

  * `Vagrantfile <../components/centos/centos-k8s-singlenode-setup/Vagrantfile>`_
  * `README <../components/centos/centos-k8s-singlenode-setup/README.rst>`_

* OpenShift Origin for use with host-based tools or via ``vagrant ssh``

  * `Vagrantfile <../components/centos/centos-openshift-setup/Vagrantfile>`_
  * `README <../components/centos/centos-openshift-setup/README.rst>`_

* Apache Mesos Marathon for use with host-based tools or via ``vagrant ssh``

  * `Vagrantfile <../components/centos/centos-mesos-marathon-singlenode-setup/Vagrantfile>`_
  * `README <../components/centos/centos-mesos-marathon-singlenode-setup/README.rst>`_

Using the box with Host-based Tools (Eclipse and CLIs)
======================================================

Many users may wish to use the ADB from their Host so it can seamlessly interact
with their files, preferred development tools, etc.

Today, the ADB exposures the docker daemon port so that tools like Eclipse and
the docker CLI can interact with it. For security reasons, the docker daemon is
TLS protected, so in addition to exposing the port you must configure the docker
command on the host to use TLS and the right port. This can be done easily with
the ``vagrant-service-manager`` Vagrant plugin. The plugin will provide you with the TLS
certificate and the proper environment variables.

1. Install the ``vagrant-service-manager`` plugin in order to get easy access to the
   correct environment variables and the TLS certificates.

   ::

       vagrant plugin install vagrant-service-manager

   More information about the vagrant-service-manager plugin is `available in the source
   repository`_.

.. _available in the source repository: https://github.com/projectatomic/vagrant-service-manager

2. Get the correct environment variables and TLS certificates for `docker` using the plugin.
   The example below shows the command and the output for Linux::

    $ vagrant service-manager env docker
    Set the following environment variables to enable access to the
    docker daemon running inside of the vagrant virtual machine:

    export DOCKER_HOST=tcp://172.13.14.1:5555
    export DOCKER_CERT_PATH=/home/foo/bar/adb/.vagrant/machines/default/virtualbox/.docker
    export DOCKER_TLS_VERIFY=1
    export DOCKER_MACHINE_NAME="90d3e96"

   *Note:* The output is similar for Mac OS X. On Microsoft Windows the
   environment is setup using the `setx` command.

   Setting these environment variables allows program, such as Eclipse and the
   docker CLI to access the docker daemon.

3. Begin developing.

   If you are using the docker CLI, you can just run it from the command line
   and it will work as expected.  If you need to download a copy of the docker
   CLI, you can find it listed as a "client binary" download in the official
   `Docker Repositories <https://github.com/docker/docker/releases>`_.

   If you are using Eclipse, you should follow these steps:

   **Note:** Testing has been done with Eclipse 4.5.0.

   1. Install the `Docker Tooling`_ plugin.

   2. Enable the three Docker Views (Docker Explorer, Docker Containers, and
      Docker Images) by choosing Windows->Show Views->Others.

   3. Enable the Console by choosing Windows->Show Views->Console.

   4. In the ``Docker Explorer`` view, click to add a connection. You should
      provide a "connection name." If your Environment Variables are set
      correctly, the remaining fields will autopopulate. If not, using the
      output from ``vagrant service-manager env docker``, put the DOCKER_HOST
      variable in the "TCP Connection" field and the DOCKER_CERT_PATH in the
      "Authentication Section" Path.

   5. You can test the connection and then accept the results. At this point,
      you are ready to use the ADB with Eclipse.

.. _Docker Tooling: http://www.eclipse.org/community/eclipse_newsletter/2015/june/article3.php

Using the box via SSH
=====================

Today most users will do their work inside the Vagrant box.  Access the box by
using ``ssh`` to login to it with the following command::

    vagrant ssh

You are now at a shell prompt inside the Vagrant box. You can now execute
commands and use the tools provided.

Using ``docker``
################

The ADB provides a full container environment and is running both ``docker`` and
``kubernetes``. All standard commands will work, for example::

   docker pull centos
   docker run -t -i centos /bin/bash

Using Atomic App and Nulecule
#############################

Details on these projects can be found at these urls:

* Atomic App: https://github.com/projectatomic/atomicapp
* Nulecule: https://github.com/projectatomic/nulecule

The `helloapache`_ example can be used to test your installation.

*Note:* Many Nulecule examples expect a working kubernetes environment. To setup
a single node kubernetes environment use the `Vagrantfile`_ and refer the
corresponding `README`_.

You can verify your environment with by executing ``kubectl get nodes``. The
expected output is::

    $ kubectl get nodes
    NAME        LABELS                             STATUS
    127.0.0.1   kubernetes.io/hostname=127.0.0.1   Ready

.. _helloapache: https://registry.hub.docker.com/u/projectatomic/helloapache/
.. _README: ../components/centos/centos-k8s-singlenode-setup/README.rst
.. _Vagrantfile: ../components/centos/centos-k8s-singlenode-setup/Vagrantfile

Destroying the Vagrant Box
==========================

Warning, this will destroy any data you have stored in the Vagrant box. You will
not be able to restart this instance and will have to create a new one using
``vagrant up``.

::

    vagrant destroy
