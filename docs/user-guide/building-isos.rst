.. _building-isos:

*************
Building ISOs
*************

Since Cobbler uses the systemd hardening option "PrivateTmp" you can't write or read files from your ``/tmp`` when you
run Cobbler via systemd as a service.

Per default this builds an ISO for all available systems and profiles.

.. note:: All systems refers to systems that are profile based. Systems with a parent image based systems will be
          skipped.

If you want to generate multiple ISOs you need to execute this command multiple times (with different ``--iso`` names).

NOTE: This feature is currently only supported for the following architectures: x86_64, ppc, ppc64, ppc64le and ppc64el.

Under the hood
##############

Under the hood the tool "xorriso" is used. It is being executed in the "mkisofs" (the predecessor) compatibility mode.
Thus we don't execute "mkisofs" anymore. Please be aware of this when adding CLI options.

On the Python side we are executing the following command:

.. code-block:: shell

   xorriso -as mkisofs $XORRISOFS_OPTS -isohybrid-mbr $ISOHDPFX_location -c isolinux/boot.cat \
     -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot \
     -e $EFI_IMG_LOCATION -no-emul-boot -isohybrid-gpt-basdat -V \"Cobbler Install\" \
     -o $ISO $BUILDISODIR

Explanation what this command is doing:

.. code-block:: shell

   xorriso -as mkisofs \
     -isohybrid-mbr /usr/share/syslinux/isohdpfx.bin \  # --> Makes the image MBR bootable and specifies the MBR File
     -c isolinux/boot.cat \                             # --> Boot Catalog -> Automatically created according to Syslinux wiki
     -b isolinux/isolinux.bin \                         # --> Boot file which is manipulated by mkisofs/xorriso
     -no-emul-boot \                                    # --> Does not run in emulated disk mode when being booted
     -boot-load-size 4 \                                # --> Size of 512 sectors to boot in no-emulation mode
     -boot-info-table \                                 # --> Store CD layout in the image
     -eltorito-alt-boot \                               # --> Allows to have more then one El Torito boot on a CD
     -e /var/lib/cobbler/loaders/grub/x64.efi \         # --> Boot image file which is EFI bootable, relative to root directory
     -no-emul-boot \                                    # --> See above
     -isohybrid-gpt-basdat \                            # --> Add GPT additionally to MBR
     -V "Cobbler Install" \                             # --> Name when the image is recognized by the OS
     -o /root/generated.iso \                           # --> Produced ISO file name and path
     /var/cache/cobbler/buildiso                        # --> Root directory for the build

Common options for building ISOs
################################

* ``--iso``: This defines the name of the built ISO. It defaults to ``autoinst.iso``.
* ``--distro``: Used to detect the architecture of the ISO you are building. Specifies also the used Kernel and Initrd.
* ``--buildisodir``: The temporary directory where Cobbler will build the ISO. If you have enough RAM to build the ISO
  you should really consider using a tmpfs for performance.
* ``--profiles``: Modify the profiles Cobbler builds ISOs for. If this is omitted, ISOs for all profiles will be built.
* ``--xorrisofs-opts``: The options which are passed to xorriso additionally to the above shown.
* ``--esp``: Explicitly specify the EFI System Partition (ESP). The default is to attempt to search for ESP, and
  generate one if an ESP cannot be found.

Building standalone ISOs
########################

have to provide the following parameters:

* ``--standalone``: If this flag is present, Cobbler will build an ISO which can be installed without network access.
* ``--airgapped``: If this flag is present, Cobbler will build an ISO which contains all mirrored repositories for
  extended installations.
* ``--source``: The directory with the sources for the image.

Building net-installer ISOs
###########################

You have to provide the following parameters:

* ``--systems``: Filter the systems you want to build the ISO for.
* ``--exclude-dns``: Flag to add the nameservers (and other DNS information) to the append line or not. This only has
  an effect in case you supply ``--systems`` and the system contains the ``--name-servers`` configuration.

Building ISOs for Secure Boot
#############################

When you build an ISO from a distribution that is stored in the Cobbler Web directory, e.g.
 ``/srv/www/cobbler/distro_mirror/``, Cobbler looks for the EFI System Partition (ESP) of the provided distribution
 automatically. This is the default behavior, for example, when you use the ``cobbler import`` command to
 create a distribution.

If you create a distribution that references files stored outside of the Cobbler Web directory, e.g.
``/usr/share/tftpboot-installation/SLE-15-SP6-x86_64``, use the ``--esp`` parameter to specify the location of
the ESP, for example:

.. code-block:: shell

   cobbler buildiso --esp="/usr/share/tftpboot-installation/SLE-15-SP6-x86_64/boot/x86_64/efi" \
     --iso=/tmp/out.iso --distro=sles15-sp6

Examples
########

Building exactly one network installer ISO for a specific profile (suitable for all underlying systems):

Building exactly one network installer ISO for a specific system:

Building exactly one airgapped installable ISO for a specific system:

Links with further information
##############################

* `xorriso homepage <https://www.gnu.org/software/xorriso/>`_
* `xorriso manpage <https://www.gnu.org/software/xorriso/man_1_xorriso.html>`_
* `mkisofs manpage <https://linux.die.net/man/8/mkisofs>`_
