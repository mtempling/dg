# dg

Automatic testing of [dmce-gui](https://github.com/PatrikAAberg/dmce-gui)
within an incus container.

## architecture

```
ove digraph
                        ┌─────────────┐
       ┌─────────────── │     ag      │
       │                └─────────────┘
       │                  │
       │                  │
       │                  ▼
       │                ┌─────────────┐
       │                │    base     │ ◀┐
       │                └─────────────┘  │
       │                                 │
  ┌────┘                                 │
  │                                      │
  │  ┌────────────┐     ┌─────────────┐  │
  │  │ screencast │ ◀── │     dg      │  │
  │  └────────────┘     └─────────────┘  │
  │                       │              │
  │                       │              │
  │                       ▼              │
  │  ┌────────────┐     ┌─────────────┐  │
  │  │   godot    │ ◀── │  dmce_gui   │  │
  │  └────────────┘     └─────────────┘  │
  │                       │              │
  │                       │              │
  │                       ▼              │
  │  ┌────────────┐     ┌─────────────┐  │
  └▶ │     xz     │ ◀── │    dmce     │ ─┘
     └────────────┘     └─────────────┘
                          │
                          │
                          ▼
                        ┌─────────────┐
                        │ clang_tools │
                        └─────────────┘
```

## setup

```
curl -sSL https://raw.githubusercontent.com/Ericsson/ove/master/setup | bash -s dg https://github.com/mtempling/dg.git
cd dg
source ove
ove create-instance base debian/12/amd64
```

The "ove create-instance..." command will first make sure that both
[incus](https://github.com/lxc/incus) and
[Xpra](https://github.com/Xpra-org/xpra) is installed on the host. NOTE: The
incus installation is just a minimal installation.

When incus and Xpra has been installed, create-instance will continue and launch an
incus container based on Debian 12. A few packages will be installed within
the container:

* openssh
* packages needed to run OVE
* Xfce
* Xpra

Finally the [post-create-instance](hooks/post-create-instance) hook script will install
a Xfce autostart file and restart the container. The autostart script will
launch the following command:

```
ove entry dg
```

Looking into the dg's [entry code](projects/dg/entry) one can see that
ag is built, then OVE is configured to screenrecord all OVE commands. Finally,
"ove dmce trace-lazy ag" is launched two times. First with Terminal GUI then
using the Godot GUI.

## useful commands

Attach to the container and peek into the action:

```
xpra attach --ssh=ssh ssh://${OVE_INSTANCE_NAME:?}
```

Find the recorded screencast:

```
find $OVE_LOG_DIR -type f -name "*-ove-dmce-${OVE_INSTANCE_NAME:?}-dg.mp4"
```

Remove the container:

```
ove delete-instance ${OVE_INSTANCE_NAME:?}
```

## configuration

Check [this file](.oveconfig).
