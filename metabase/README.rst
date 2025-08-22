=====================
Metabase Rootless Quadlet
=====================

------------
Installation
------------ 

------------------
System Integration
------------------


- Copy the follwing files to the following directory: `/etc/containers/systemd/users/$(UID)` where `UID`
  is the user ID of the user that runs the quadlet.
  The configuration files should be in the `/etc/metabase/` directory. A the time of this writing the only
  file we're using is the `container.env` file.

- For rootless containers (do someone sane is running containers as root?) we need to generate
  the service file because quadlet does not honor the `User` and `Group` directives [1]

  If needed copy the metabase* files and the container.env files to a remote server.

  .. code-block:: bash

      root@localhost ~# scp metabase* server:.
      root@localhost ~# scp container.env server:.

  .. code-block:: bash

      root@server ~# groupadd metabase
      root@server ~# useradd -g metabase -s /bin/bash -d /home/metabase -c "Metabase User" metabase
      root@server ~# cp metabase.container metabase.volume \
        /etc/containers/systemd/
      root@server ~# mkdir /etc/metabase /var/lib/metabase
      root@server ~# cp container.env /etc/metabase/
      root@server ~# chown -R metabase:metabase /etc/metabase/ /var/lib/metabase

  For testing purpouses the `~/.config/containers/systemd` directory can be used but it is not recommended for
  production because the visibility of this user directories is not evident:

  .. code-block:: bash

      root@server ~me gustó metabase# mkdir  ~/.config/containers/systemd
      root@server ~# cp metabase.container metabase.network metabase.volume \ 
        ~/.config/containers/systemd/
      root@server ~# mkdir ~/.config/metabase
      root@server ~# cp container.env .config/containers/systemd/
      root@server ~# sed -i 's/EnvironmentFile=\/etc\/metabase\/container.env/EnvironmentFile=container.env/' \
                     ~/.config/containers/systemd/metabase.container

- Enable user systemd units to be started without a login session


  .. code-block:: bash

      root@server ~# loginctl enable-linger 1000


- Load and enable the systemd quadlets

  - System wide:

  .. code-block:: bash

      root@server ~# systemctl daemon-reload
      root@server ~# systemctl enable metabase

  - For user setup:

  .. code-block:: bash

      metabase@server ~$ systemctl --user daemon-reload
      metabase@server ~$ systemctl --user enable metabase


Due a systemd bug quadlets running as user can't create the proper cidfile.
Quadlet creates the option: `--cidfile=%t/%N.cid` and does not offer the option to
set it to a proper path (eg .--cidfile=%t/user/metabase/%N.cid)

  .. code-block:: bash

      root@server ~# cd /etc/systemd/system
      root@server system # /usr/libexec/podman/quadlet --dryrun > metabase.service
      root@server system # cd /etc/containers/systemd
      root@server system # mv metabase.container metabase.container.DISABLED
      root@server system # mv metabase.network metabase.network.DISABLED
      root@server system # mv metabase.volume metabase.volume.DISABLED

- Start the service

  - System wide:

  .. code-block:: bash

      metabase@server ~$ systemctl --user start metabase

   - For user suetup:

  .. code-block:: bash

      metabase@server ~$ systemctl --user start metabase

---------------
Troubleshooting
---------------

- To debug the quadlet add the `GlobalArgs=--log-level=debug` to the `[Container]` section in the
  `metabase.container` file.


  .. code-block:: bash

   ...
   [Container]
   GlobalArgs=--log-level=debug
   ...

====
TODO
====

Add instructions for using a normal Postgres server


