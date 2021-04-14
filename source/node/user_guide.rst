.. SPDX-FileCopyrightText: 2020 Hyperledger
   SPDX-License-Identifier: CC-BY-4.0

.. _User guide:

User guide
==========

Perun-node provides an interface for users to use and manage multiple state
channels across their complete life cycle.

It can be used as a:

  1. **Stand-alone application**
     
     - This is intended to add state channel functionality to third-party
       applications that have insufficient resources to run the perun-node
       software locally e.g., constrained embedded devices.

     - In order to quickly try the using the client API, you can use the
       ``peurunnodecli`` application. This serves as a reference implementation
       for using the client stubs.

     - You can directly integrate the client stubs into you application by
       generating it from the proto buff definition and using it your
       application.

  2. **Library.**

     - This is intended to add state channel functionality to third-party
       applications that can run the perun-node software locally.

     - You can import the necessary packages from perun-node go module into
       your application and directly use the method, function calls.


The next section describes the pre-requisites common for both these methods.
After reading that, you can proceed to the guide covering your choice of usage. 


.. Run tests for perun-node
.. ------------------------

.. Before using the perun-node, we can ensure if it is working on your machine by
.. running the tests.

.. 1. Open a terminal, clone the project repository and switch into the project
..    directory.

.. .. code-block::

..   $ git clone https://github.com/hyperledger-labs/perun-node.git
..   $ cd perun-node

.. 2. Start ganache-cli as mentioned in :ref:`Using ganache-cli` section in a
..    separate terminal.

   
.. 3. Run the tests.

.. .. code-block::

..   $ go tests -tags=integration -count=1 -p 1 ./...

.. Using the perun-node
.. --------------------

.. **Configuration**

.. To use the perun-node, we first need to specify the configuration for
.. initializing the node. This includes

.. - Parameters for connecting with the blockchain network.
.. - Addresses of perun adjudicator and asset holder contracts.
.. - List of adapters supported for off-chain communication, ID provider and
..   currency parsing.

.. Once the node is initialized, we need to open a session. Within a session, an
.. instance of wallet provider, ID provider are initialized. For this, we need to
.. specify the following configuration data:

.. - (optional) Parameters for connecting with the blockchain network.
.. - (optional) Addresses of adjudicator and asset holder contracts.
.. - Parameters for connecting with the wallet provider.
.. - Parameters for connecting with ID provider.
.. - Parameters for accessing persistence database.

.. .. note::

..   If the optional parameter values are provided in session config, then it
..   will supersede the values provided in node config.

.. .. note::

..   In the tutorial, we will be using
..   - A local file as ID provider.
..   - Ethereum keystore as wallet provider.
..   - levelDB as persistence database.

.. For steps to generate the configuration and doing a complete cycle of payment
.. channel transaction, refer to the corresponding section depending upon your
.. choice of usage.

.. toctree::
   :maxdepth: 1

   user_guide_pre_requisites
   user_guide_setting_up_blockchain

   user_guide_stand_alone
