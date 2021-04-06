.. SPDX-FileCopyrightText: 2020 Hyperledger
   SPDX-License-Identifier: CC-BY-4.0


Using perun node as a stand-alone software.
===========================================

We can use perun-node as a stand-alone software by starting an instance of
perun-node and then accessing the remote API by using client stubs. For this,
we will need,

**1. perunnode executable**

We can run the ``make`` command to generate the ``perunnode`` executable
file. Running the executable without any arguments will print the help
message.

.. code::

    $ # From the project directory.
    $ ./perunnode

The make command will also be generate another executable ``perunnodecli``,
which will be using later.

**2. Configuration artifacts**

We can run the ``perunnode`` executable with ``generate`` command to generate
the configuration artifacts.

.. code::

    $ ./perunnode generate

This will generate

- Node configuration: node.yaml file.
- Session configuration: Two directories (alice and bob) each containing
  ``session.yaml`` file, ``idprovider.yaml`` file and ``keystore`` directory
  with keys corresponding to the on-chain and off-chain accounts.

If you are using ganache-cli node and have started it using the command as
mentioned in the :ref:`Using ganache-cli` section, then the configuration
artifacts can be used as such. If not, then you need to update the parameter
accordingly.

**3. perunnodecli executable**

``perunnodecli`` is an interactive command line application that uses gRPC
client stubs to interact with perunnode via gRPC interface. We had already
generated this executable when running the ``make`` command in step 1.

Running the command without any argument will start an interact shell.

.. code::

    $ # From the project directory.
    $ ./perunnodecli

.. note::
  For the code blocks that appear in consequent sections, if they are
  prefixed by

  - **$**, execute them in terminal,
  - **>**, execute them in perun-node cli.

Deploying the contracts
-----------------------

This steps needs to be done only once after starting the ganache-cli node.
We will be using the helper function in ``perunnodecli`` to do this.

Start the ``perunnodecli`` interactive shell, run the following two commands to
deploy the contracts, and then exit the shell.

.. code::

  $ ./perunnodecli
  > chain set-blockchain-address ws://127.0.0.1:8545
  > chain deploy-perun-contracts
  > exit
  $

Starting the perunnode
----------------------

Run the below command to start the perun-node,

.. code-block::

  $ ./perunnode run --configfile "node.yaml"

Once it is running, the node configution will be printed. Leave this running in
the current terminal.
.. code-block::

  $ ./perunnode run
  Using node config file - node.yaml
  Running perun node with the below config:
  {LogLevel:             "debug",
   LogFile:              "",
   ChainURL:             "ws://127.0.0.1:8545",
   Adjudicator:          "0x9daEdAcb21dce86Af8604Ba1A1D7F9BFE55ddd63",
   Asset:                "0x5992089d61cE79B6CF90506F70DD42B8E42FB21d",
   ChainConnTimeout:     10s,
   OnChainTxTimeout:     1m0s,
   ResponseTimeout:      10s,
   CommTypes:            ["tcp"],
   IDProviderTypes:      ["local"],
   CurrencyInterpreters: ["ETH"]}.

  Serving payment channel API via grpc at port :50001
.. note::

  If log file was set to empty string (default value), the logs will be printed
  in this terminal.


Initializing perunnode-cli
--------------------------

We will now initialize two instances of ``perunnodecli``, one each for alice
and bob.

1. Open two new terminals side by side and run the ``perunnodecli`` command in
   each of them.

.. note::
  To see a complete list of commands, type ``help``. To see the list of
  sub-commands for each command, type the command without any arguemnts and
  hit enter.

  All commands and sub commands support autocompletion.

2. Set the blockchain address. This address will be used by the sub-commands of
   chain command to read on-chain balance.

.. code-block::

  > chain set-blockchain-address ws://127.0.0.1:8545

.. note::
  The chain command is not a part of perun-node API. It is a helper command in
  perun-node cli to directly interact with blockchain.

4. Read the on-chain balance. The addresses for default configuration are
   available as auto-complete suggestion, if some other address was used, it
   needs to be entered manually.

.. code-block::

  # Alice's default on-chain address.
  > chain get-on-chain-balance 0x8450c0055cB180C7C37A25866132A740b812937B

  # Bob's default on-chain address.
  > chain get-on-chain-balance 0xc4bA4815c82727554e4c12A07a139b74c6742322

You can use these commands at any time before opening, while open or after
closing a payment channel.

Open a session; open, use and close a payment channel
-----------------------------------------------------

From here on, choose one terminal for alice role and one for bob role. In each
step, a comment about the command will contain the role. If no role is
mentioned above a command, it can be typed into any of the terminals.

1. Open a session each for Alice and Bob. Also check if the other
   participant's peer IDs are registered.

.. code-block::

  > # [In Alice's CLI]
  > node connect :50001
  > session open alice/session.yaml
  > peer-id get bob

.. code-block::

  > # [In Bob's CLI]
  > node connect :50001
  > session open bob/session.yaml
  > peer-id get alice

.. note::
  Getting the peer ID will also add the peer alias to auto-completion list.
  When you press ``TAB`` after the sub-commands of ``channel``, ``payment``
  command that expect ``peer alias`` as the first argument, these aliases
  will be suggested.

2. Send a payment channel opening request and accept it.

a. Alice sends a payment channel opening request to Bob.

.. code-block::

  > # [In Alice's CLI]
  > channel send-opening-request bob 1 2

b. Bob receives a notification in his CLI. The incoming request contains a
   request ID.

.. code-block::

  > # [In Bob's CLI]
  > Channel opening request notification received. Notification Alias: request_1_alice.
  Currency: ETH, Balance: [alice:1.000000 self:1.000000].
  Expires in 10s.

.. note::
  Request ID is an identifier assigned by perun-node cli application to
  reference the request when accepting/rejecting it.

c. Bob accepts the request before it expires.

.. code-block::

  > # [In Bob's CLI]
  > channel accept-opening-request request_1_alice

d. Once Bob accepts the request, the channel will be funded on-chain. Once it
   is opened, both Alice and Bob receive notifications in their CLIs.

.. code-block::

  > # [In Alice's CLI]
  > Channel opened. Alias: ch_1_bob.
  ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f, Currency: ETH, Version: 0, Balance [bob:1.000000 self:1.000000].

  Subscribed to payment notifications on channel ch_1_bob (ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f)

.. code-block::

  > # [In Bob's CLI]
  > Channel opened. Alias: ch_1_alice.
  ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f, Currency: ETH, Version: 0, Balance [alice:1.000000 self:1.000000]

  Subscribed to payment notifications on channel ch_1_bob (ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f)


.. note::

  Channel alias is an identifier assigned by perun-node cli application to
  reference the channel when sending/receiving payments or closing it. It will
  be different for alice and bob, as it local to the perun-node cli instance.

  In this case, it is ``ch_1_bob`` for ``Alice`` and ``ch_1_alice`` for ``Bob``.

3. List all the open channels in a session along with their latest state.

.. code-block::

  > channel list-open-channels

3. Send a payment channel opening request and reject it.

Repeat sections (a) and (b) as in the step 2. This time, the request ID will be
different: ``request_alice_2``. Instead of accepting the request, reject it.

c. Bob rejects the request before it expires.

.. code-block::

  > # [In Bob's CLI]
  > channel reject-opening-request request_2_alice

d. After the channel is rejected, Bob will get the following response.

.. code-block::

  > Channel proposal rejected successfully.

e. And, Alice will get the following response.

.. code-block::

  > Error opening channel : The request was rejected by peer

4. Send a payment on the open channel and accept it.

a. Alice sends a payment to bob on the open channel (ch_1_bob).

.. code-block::

  > # [In Alice's CLI]
  > payment send-to-peer ch_1_bob 0.1


b. Bob receives a notification. Note that, the proposed version is different
   from the current version.

.. code-block::

  > # [In Bob's CLI]
  > Payment notification received on channel ch_1_alice. (ID:94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f)
  Current:        Currency: ETH, Balance: [alice:1.000000 self:1.000000], Version: 0.
  Proposed:       Currency: ETH, Balance: [alice:0.900000 self:1.100000], Version: 1.
  Expires in 10s.


c. Bob accepts the payment.

.. code-block::

  > # [In Bob's CLI]
  > payment accept-payment-update-from-peer ch_1_alice

d. Once the payment is accepted, both Alice and Bob receive channel update
   notifications.

.. code-block::

  > # [In Alice's CLI]
  > Payment sent to peer on channel ch_1_bob. Updated channel Info:
  ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f, Currency: ETH, Version: 1, Balance [bob:1.100000 self:0.900000].

.. code-block::

  > # [In Bob's CLI]
  > Payment channel updated. Alias: ch_1_alice. Updated Info:
  ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f, Currency: ETH, Version: 1, Balance [alice:0.900000 self:1.100000]


5. Send a payment on the open channel and reject it.

Repeat sections (a) and (b) as in the above command. This time the current and
proposed versions will be different. Instead of accepting the channel, reject
it.

c. Bob rejects the payment from Alice.

.. code-block::

  > # [In Bob's CLI]
  > payment reject-payment-update-from-peer ch_1_bob

d. Once the payment is rejected, Bob will get the following response.

.. code-block::

  > # [In Bob's CLI]
  > Payment channel update rejected successfully.

e. And Alice will get the following response.

.. code-block::

  > # [In Alice's CLI]
  > Error sending payment to peer: The request was rejected by peer.

6. Error on closing a session with open channels without force option.

   Closing a session with open channels without ``force`` option should return
   an error. It is not safe to do so because, the node will not be able to
   refute if the other participant registers and finalizes an older state on
   the blockchain.

.. code-block::

  > session close no-force
  > Error closing session : Session cannot be closed (without force option) as there are open channels

7. Close the channel.

a. Alice sends a request to close the channel. Perun-node will send a channel
update to Bob, marking the latest state as final.

.. code-block::

  > # [In Alice's CLI]
  > channel close-n-settle-on-chain ch_1_bob

.. note::

  **Collaborative and Non collaborative channel close:**

  When any one of the channel participants sends a channel close request to the
  perun-node, an update is send to other participants marking the latest state
  of the channel as final.

  If this update is accepted by the peer, then this is called finalized
  state. A finalized state can be registered on the blockchain in a single
  transaction and the funds can be withdrawn immediately. This is called
  **Collaborative close**.

  If this update is rejected by the peer, then the latest state of channel is
  registered on the blockchain and both parties will have to wait for the
  challenge duration to pass. Once it passes, the state should be concluded
  on the blockchain and then the paricipants can withdraw their funds. This
  is called **Non collaborative close**.


b. Bob gets a finalizing channel update.

.. code-block::

  > # [In Bob's CLI]
  > Finalizing payment notification received on channel ch_1_alice. (ID:94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f)
  Channel will closed if this payment update is responded to.
  Current:        Currency: ETH, Balance: [alice:0.900000 self:1.100000], Version: 1.
  Proposed:       Currency: ETH, Balance: [alice:0.900000 self:1.100000], Version: 2.
  Expires in 10s.

c. Bob accepts the notification, thereby enabling collaborative close.

.. code-block::

  > # [In Bob's CLI]
  > payment accept-payment-update-from-peer ch_1_alice

d. Once Bob accepts the update, he gets the following response. In the
   background, finalized state will be registered on the blockchain.

.. code-block::

  > # [In Bob's CLI]
  > Payment channel updated. Alias: ch_1_alice. Updated Info:
  ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f, Currency: ETH, Version: 2, Balance [alice:0.900000 self:1.100000]

e. Once the finalized state is registered on the blockchain, funds will be
   withdrawn for both the participants. Both Alice and Bob will receive channel
   close notifications.

.. code-block::

  > # [In Alice's CLI]
  > Payment channel close notification received on channel ch_1_bob (ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f)
  Currency: ETH, Balance: [bob:1.100000 self:0.900000], Version: 2.

.. code-block::

  > # [In Bob's CLI]
  > Payment channel close notification received on channel ch_1_alice (ID: 94241c75d058186f40826be7ae0803f7731a423903c494faa05b347443bb0a4f)
  Currency: ETH, Balance: [alice:0.900000 self:1.100000], Version: 2.


8. Close the session:

Since the open channels are closed, the session can be closed with the same
command as in step 6. This should return a success response as shown below.

.. code-block::

  > # [In Alice's CLI]
  > session close no-force

  > # [In Bob's CLI]
  > session close no-force

9. Try out persistence of channels:

a. Open a session each for alice and bob by following step 1. Then open a few
   channels by following step 2.  Now close the session for alice and bob with
   ``force`` option.

.. code-block::

  > # [In Alice's CLI]
  > session close force

  > # [In Bob's CLI]
  > session close force

b. Now, in the same or different instances of CLI, open the sessions for alice
   and bob specifying the same configuration file. The session will be opened,
   channels restored from persistence and their latest state along with the
   aliases will be printed. Now, you can send/receive payments on these
   channels and close them.

Remarks:

1. You can try to open as many channels as desired using the commands as
   described in step 2. Each channel is addressed by its alias (that will be
   suggested in auto-complete).

2. You can also try and send as many payments as desired using the commands as
   described in step 4. However, whenever a new payment notification is
   received, the previous one is automatically dropped. This however, is not a
   feature of payment channel API, where you can respond to any of the
   notifications as long as they have not expired. It was just a feature in the
   perunnode-cli app to make it simpler.


3. The purpose of the perunnode-cli software is to demo the payment channel API
   and also as a reference implementation for using the gRPC client stubs. In
   order to integrate the client for perun-node with your application, you can
   generate the gRPC client stubs in the desired language and directly use them
   in your application.
