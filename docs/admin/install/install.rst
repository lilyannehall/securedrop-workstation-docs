Installing SecureDrop Workstation
=================================

Copy the submission key
~~~~~~~~~~~~~~~~~~~~~~~

In order to decrypt submissions, your SecureDrop Workstation will need a copy of the secret key from your SecureDrop instance's SVS.

.. note::
   Secret submission keys that are password-protected will need to have their password removed in order for SecureDrop Workstation to function properly. To export a copy that does not require a passphrase, see :doc:`/admin/reference/removing_gpg_passphrase`.

To protect this key and preserve the air gap, you will need to connect the SVS USB to a Qubes VM with no network access, and copy it from there to ``dom0``. Note that you cannot directly copy and paste to the ``dom0`` VM from another VM - instead, follow the steps below to copy the file into ``dom0``:

- First, use the network manager widget in the upper right panel to disable your network connection. These instructions refer to the ``vault`` VM, which has no network access by default, but if the SVS USB is attached to another VM by mistake, this will offer some protection against exfiltration.

- Next, choose **Q > Apps > vault > Thunar File Manager** to open the file manager in the ``vault`` VM.

- Connect the SVS USB to a USB port on the Qubes computer, then use the devices widget in the upper right panel to attach it to the ``vault`` VM. There will be three entries for the USB in the section titled **Data (Block) Devices**. Choose the *unlabeled* entry (*not* the one labeled "TAILS") annotated with a ``sys-usb`` text that ends with a number, like ``sys-usb:sdb2``. That is the persistent volume.

  |Attach TailsData|

- In the the ``vault`` file manager, select the persistent volume's listing in the lower left sidebar. It will be named ``N GB encrypted``, where N is the size of the persistent volume. Enter the SVS persistent volume passphrase to unlock and mount it. When asked if you would like to forget the password immediately or remember it until you logout, choose the option to **Forget password immediately**.

  .. note::

    You will receive a message that says **Failed to open directory "TailsData"**. This is normal behavior and will not cause any issues with the subsequent steps.

  |Unlock TailsData|

- Open a ``dom0`` terminal by opening the **Q Menu**, selecting the gear icon on the left-hand side, then selecting **Other > Xfce Terminal**. Once the Terminal window opens, run the following command to list the SVS submission key details, including its fingerprint:

  .. code-block:: sh

    qvm-run --pass-io vault \
      "gpg --homedir /run/media/user/TailsData/gnupg -K --fingerprint"

- Next, run the comand:

  .. code-block:: sh

    qvm-run --pass-io vault \
      "gpg --homedir /run/media/user/TailsData/gnupg --export-secret-keys --armor <SVSFingerprint>" \
      > /tmp/sd-journalist.sec

  where ``<SVSFingerprint>`` is the submission key fingerprint, typed as a single unit without whitespace. This will copy the submission key in ASCII format to a temporary file in dom0, ``/tmp/sd-journalist.sec``.

- Verify the that the file starts with ``-----BEGIN PGP PRIVATE KEY BLOCK-----`` using the command:

  .. code-block:: sh

    head -n 1 /tmp/sd-journalist.sec

- In the ``vault`` file manager, right-click on the **TailsData** sidebar entry, then select **Unmount** and disconnect the SVS USB.


.. _copy_journalist:

Copy *Journalist Interface* details
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SecureDrop Workstation connects to your SecureDrop instance's API via the *Journalist Interface*. In order to do so, it will need the *Journalist Interface* address and authentication info. As the clipboard from another VM cannot be copied into ``dom0`` directly, follow these steps to copy the file into place:

- Locate an *Admin Workstation* or *Journalist Workstation* USB drive. Both hold the address and authentication info for the *Journalist Interface*; if you also want to copy the journalist user's password database, use the *Journalist Workstation* USB drive.

- Connect the USB drive to a USB port on the Qubes computer, then use the devices widget in the upper right panel to attach it to the ``vault`` VM. There will be 3 listings for the USB in the widget: one for the base USB, one for the Tails partition on the USB, labeled ``Tails``, and a 3rd unlabeled listing, for the persistent volume. Choose the third listing.

- In the the ``vault`` file manager, select the persistent volume's listing in the lower left sidebar. It will be named ```N GB encrypted``, where N is the size of the persistent volume. Enter the persistent volume passphrase to unlock and mount it. When prompted, select the option to **Forget password immediately**.

- Copy the *Journalist Interface* configuration file to ``dom0``. If your SecureDrop instance uses v3 onion services, use the following command:

  .. code-block:: sh

    qvm-run --pass-io vault \
      "cat /run/media/user/TailsData/Persistent/securedrop/install_files/ansible-base/app-journalist.auth_private" \
      > /tmp/journalist.txt

- Verify that the ``/tmp/journalist.txt`` file on ``dom0`` contains valid configuration information using the command ``cat /tmp/journalist.txt`` in the ``dom0`` terminal.

- If you used an *Admin Workstation* USB drive, or you don't intend to copy a password database to this workstation, safely disconnect the USB drive now. In the ``vault`` file manager, right-click on the **TailsData** sidebar entry, then select **Unmount** and disconnect the USB drive.

Copy SecureDrop login credentials
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Users of SecureDrop Workstation must enter their username, passphrase and two-factor code to connect with the SecureDrop server. You can manage these passphrases using the KeePassXC password manager in the ``vault`` VM. If this laptop will be used by more than one journalist, we recommend that you shut down the ``vault`` VM now (using the Qube widget in the upper right panel), skip this section, and use a smartphone password manager instead.

In order to set up KeePassXC for easy use:

- Add KeePassXC to the application menu by selecting it from the list of available apps in **Q > Apps > vault > Settings > Applications** and pressing the button labeled **>** (do not press the button labeled **>>**, which will add *all* applications to the menu).

- Launch KeePassXC via **Q > Apps > vault > KeePassXC**. When prompted to enable automatic updates, decline. ``vault`` is networkless, so the built-in update check will fail; the app will be updated through system updates instead.

- Close the application.

.. important::

   The *Admin Workstation* password database contains sensitive credentials not required by journalist users. Make sure to copy the credentials from the *Journalist Workstation* USB.

In order to copy a journalist's login credentials:

- If a *Journalist Workstation* USB is not currently attached, connect it, attach it to the ``vault`` VM, open it in the file manager, and enter its encryption passphrase.

- Locate the password database. It should be in the ``Persistent`` directory, and will typically be named ``keepassx.kdbx`` or similar.

- Open a second ``vault`` file manager window (``Ctrl + N`` in the current window) and navigate to the **Home** directory.

- Drag and drop the password database to copy it.

- In the ``vault`` file manager, right-click on the **TailsData** sidebar entry, then select **Unmount** and disconnect the *Journalist Workstation* USB. Close this file manager window.

- In the file manager window that displays the home directory, open the copy you made of the password database by double-clicking it.

- If the database is passwordless, KeePassXC may display a security warning when opening it. To preserve convenient passwordless access, you can protect the database using a key file, via **Database > Database settings > Security > Add additional protection > Add Key File > Generate**. This key file has to be selected when you open the database, but KeePassXC will remember the last selection.

- Inspect each section of the password database to ensure that it contains only the information required by the journalist user to log in.

- Close the application window and shut down the ``vault`` VM (using the Qube widget in the upper right panel).

.. _download_rpm:

Download and install SecureDrop Workstation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With the key and configuration available in ``dom0``, you're ready to set up SecureDrop Workstation:

- First, re-enable the network connection using the network manager widget.

- Next, start a terminal in the network-attached ``work`` VM, via **Q > Apps > work > Xfce Terminal**.

.. note:: As the next steps include commands that must be typed exactly, you may want to open a browser in the ``work`` VM, open this documentation there, and copy-and-paste the commands below into your ``work`` terminal. Note that due to Qubes' default security settings you will *not* be able to paste commands into your ``dom0`` terminal. The ``work`` browser can be opened via **Q > Apps > work > Firefox**

- In the ``work`` terminal, run the following commands to download and add the SecureDrop signing key, which is needed to verify the SecureDrop Workstation package:

  .. code-block:: sh

    gpg --keyserver hkps://keys.openpgp.org --recv-key \
      "2359 E653 8C06 13E6 5295 5E6C 188E DD3B 7B22 E6A3"

    gpg --armor --export 2359E6538C0613E652955E6C188EDD3B7B22E6A3 \
      > securedrop-release-key.pub

    sudo rpmkeys --import securedrop-release-key.pub

- In the ``work`` terminal, open a text editor with escalated privileges (for example, with the command ``sudo nano``) and create a file ``/etc/yum.repos.d/securedrop-temp.repo`` with the following contents:

  .. code-block:: none

    [securedrop-workstation-temporary]
    enabled=1
    baseurl=https://yum.securedrop.org/workstation/dom0/f37
    name=SecureDrop Workstation Qubes initial install bootstrap

- Download the SecureDrop Workstation config package to the curent working directory with the command:

  .. code-block:: sh

    dnf download securedrop-workstation-dom0-config

  Note the release version number in the filename, you'll need it below. During the download, you may be prompted to confirm importing the Qubes OS Release 4 Signing Key. You can safely do so; it will not be used during the subsequent steps.

- Verify the package with the following command:

  .. code-block:: sh

    rpm -Kv securedrop-workstation-dom0-config-<versionNumber>-1.fc37.noarch.rpm

  where ``<versionNumber>`` is the release version number you noted above. The command output should match the following text:

  .. code-block:: none

    securedrop-workstation-dom0-config-<versionNumber>-1.fc37.noarch.rpm:
      Header V4 RSA/SHA512 Signature, key ID 7b22e6a3: OK
      Header SHA256 digest: OK
      Header SHA1 digest: OK
      Payload SHA256 digest: OK
      MD5 digest: OK


- If the package verification was successful, in the ``dom0`` terminal, run the following command to transfer the RPM package to dom0:

  .. code-block:: sh

    qvm-run --pass-io work \
      "cat /home/user/securedrop-workstation-dom0-config-<versionNumber>-1.fc37.noarch.rpm" \
      > securedrop-workstation.rpm

- Verify that the RPM was transferred correctly by running the following commands:

  - in the ``work`` terminal:

    .. code-block:: sh

      sha256sum securedrop-workstation-dom0-config-<versionNumber>-1.fc37.noarch.rpm

  - in the ``dom0`` terminal:

    .. code-block:: sh

      sha256sum securedrop-workstation.rpm

  If the hash output for both files matches, the RPM was transferred successfully.

- Install the RPM using the following command in the ``dom0`` terminal:

    .. code-block:: sh

      sudo dnf install securedrop-workstation.rpm

  When prompted, type **Y** and **Enter** to install the package.

- Shut down the ``work`` VM using the Qube widget in the top-right panel.

Configure SecureDrop Workstation (estimated wait time: 60-90 minutes)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before setting up the set of VMs used by SecureDrop Workstation, you must configure the *Journalist Interface* connection and submission key.

- To add the submission key, run the following command in the ``dom0`` terminal:

  .. code-block:: sh

    sudo cp /tmp/sd-journalist.sec /usr/share/securedrop-workstation-dom0-config/

- Your submission key has a unique fingerprint required for the configuration. Obtain the fingerprint by using this command:

  .. code-block:: sh

    gpg --with-colons --import-options import-show --dry-run --import /tmp/sd-journalist.sec

  The fingerprint will be on a line that starts with ``fpr``. For example, if the output included the line ``fpr:::::::::65A1B5FF195B56353CC63DFFCC40EF1228271441:``, the fingerprint would be the character sequence ``65A1B5FF195B56353CC63DFFCC40EF1228271441``.

- Next, create the SecureDrop Workstation configuration file:

  .. code-block:: sh

    cd /usr/share/securedrop-workstation-dom0-config
    sudo cp config.json.example config.json

- The ``config.json`` file must be updated with the correct values for your instance. Open it with root privileges in a text editor such as ``vi`` or ``nano`` and update the following fields' values:

  - **submission_key_fpr**: use the value of the submission key fingerprint as displayed above
  - **hidserv.hostname**: use the hostname of the *Journalist Interface*, including the ``.onion`` TLD
  - **hidserv.key**: use the private v3 onion service authorization key value
  - **environment**: use the value ``prod``

.. note::

   You can find the values for the **hidserv.*** fields in the ``/tmp/journalist.txt`` file that you created in ``dom0`` earlier.
   The file will be formatted as follows:

   .. code-block:: none

     ONIONADDRESS:descriptor:x25519:AUTHTOKEN

- Verify that the configuration is valid using the command below in the ``dom0`` terminal:

  .. code-block:: sh

    sdw-admin --validate

- Configure infinite scrollback for your terminal via **Edit > Preferences > General > Unlimited scrollback**. This helps to ensure that you will be able to review any error output printed to the terminal during the installation.

- Finally, in the ``dom0`` terminal, run the command:

  .. code-block:: sh

    sdw-admin --apply

This command will take a considerable amount of time and approximately 4GB of bandwidth, as it sets up multiple VMs and installs supporting packages. When the command finishes, reboot the machine to complete the installation. Your SecureDrop Workstation is finally ready to use!

Test the Workstation
~~~~~~~~~~~~~~~~~~~~

To start the SecureDrop Client, double-click the SecureDrop desktop icon that was set up by the previous command. The preflight updater will start and check for updates. The system should be up-to-date and no updates should be required, but if updates are available follow the instructions in the preflight updater to apply them.

Once the update check is complete, the SecureDrop Client will launch. Log in using an existing journalist account and verify that sources are listed and submissions can be downloaded, decrypted, and viewed.

.. _Password Management Section:

Enable password copy and paste
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If you use KeePassXC in the ``vault`` VM to manage login credentials, you can enable the user to copy passwords to the SecureDrop Client using inter-VM copy and paste. While this is relatively safe, we recommend reviewing the section :doc:`Managing Clipboard Access <../reference/managing_clipboard>` of this guide, which goes into further detail on the security considerations for inter-VM copy and paste.

The password manager runs in the networkless ``vault`` VM, and the SecureDrop Client runs in the ``sd-app`` VM. To permit this one-directional clipboard use, issue the following command in ``dom0``:

.. code-block:: sh

   qvm-tags vault add sd-send-app-clipboard

Confirm that the tag was correctly applied using the ``ls`` subcommand:

.. code-block:: sh

   qvm-tags vault ls

To revoke this configuration change later or correct a typo, you can use the ``del`` subcommand, e.g.:

.. code-block:: sh

   qvm-tags vault del sd-send-app-clipboard
   
.. |Attach TailsData| image:: images/attach_usb.png
  :width: 100%
.. |Unlock Tailsdata| image:: images/unlock_tails_usb.png
  :width: 100%
