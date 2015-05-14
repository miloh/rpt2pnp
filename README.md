Pick-and-place assembly G-Code
==============================

Right now, this code translates a KiCAD RPT file to G-Code for pick-and-place
assembly on a 3D printer. You need vacuum needle and solenoids.

(this is based on https://github.com/hzeller/rpt2paste and will eventually be merged
 to one program useful for everything.)

General setup
-------------

Currently, the setup for rpt2pnp is relatively simple: you place the board on
the bed, and the tapes with opened 'lids' straight on the same bed. You tell
rpt2pnp where the board and the tapes are, and it generates G-Code to do the
operation.

There is not yet the concept of a feeder.

Using
-----

In general, you invoke `rpt2pnp` with an option to tell what to do and a
KiCAD rpt file.

To do the pick-and-place operation, you need a configuration file with the origin
of the board and the locations of the tapes. Given an rpt file, `rpt2pnp` can
generate a template configuration that you need to modify.

The invocation without parameters shows the usage:

     Usage: ./rpt2pnp <options> <rpt-file>
     Options:
        -h      : Create homer input from rpt
        -t      : Create config template from rpt to stdout. Needs editing.
     [Operations]
        -c <config> : Use long config from -t
        -C <config> : Use homer config created via homer from -h
        -p      : Pick'n place. Requires a config and rpt.
        -P      : Output as PostScript.

So a manual workflow would typically be

     $ ./rpt2pnp -t mykicadfile.rpt > config.txt

Now you need to edit the configuration to get the locations right. Then with
    
     $ ./rpt2pnp -c config.txt -p mykicadfile.rpt > pick-n-place.gcode

You generate the gcode that you can send to your machine.

If you want to create the configuration with a different program
(e.g. https://github.com/jerkey/homer ), then use the `-h` option to create
a homer template

     $ ./rpt2pnp -h mykicadfile.rpt > homer-input.txt

.. Then create a configuration with homer, and input it via the `-C` option

     $ ./rpt2pnp -C config.txt -p mykicadfile.rpt > pick-n-place.gcode

Configuration
-------------

This describes the 'long' configuration created with `-t`.
The configuration file consists of

   - Board section. Describes board and its origin. (TODO: give sample
     component positions)
   - Tape section: Describing the tapes that carry components, uniquely
     identified by `<footprint>@<component>` (e.g. `SMD_Packages:SMD-0805@2.2k`).
     Each tape has an origin and a spacing describing how far components are
     apart.

The template output creates a configuration including descriptions; you need
to modify all the numbers to match what you have on the bed.
It typically looks like this:

     Board:
     origin: 100 100 # x/y origin of the board
     
     # This template provides one <footprint>@<component> per tape,
     # but if you have multiple components that are indeed the same
     # e.g. smd0805@100n smd0805@0.1uF, then you can just put them
     # space delimited behind each Tape:
     #   Tape: smd0805@100n smd0805@0.1uF
     # Each Tape section requires
     #   'origin:', which is the (x/y/z) position of
     # the top of the first component (z: pick-up-height). And
     #   'spacing:', (dx,dy) to the next one
     #
     # Also there are the following optional parameters
     #angle: 0     # Optional: Default rotation of component on tape.
     #count: 1000  # Optional: available count on tape

     Tape: Capacitors_SMD:c_0805@C
     origin:  10 20 2 # fill me
     spacing: 4 0   # fill me
     
     Tape: SMD_Packages:SM0805@2.2k
     origin:  10 20 2 # fill me
     spacing: 4 0   # fill me

G-Code
------
Right now, the G-Code for processing steps is hardcoded in constant strings in
`gcode-picknplace.cc`.

Shortcomings
------------
Numerous. To be addressed soon.

So far only tested with a very simple setup. One thing missing
for instance is configuration of board-thickness; right now that is hardcoded
in `#define TAPE_TO_BOARD_DIFFZ`, but it really should be a parameter in the
'Board' part of the configuration.

Also, the 'origin' of the board is somewhat useless unless you really know where
that is. Better would be to give the positions of a few select components on
the board (much easier to spot with a camera), and have rpt2pnp calculate the
origin (and possibly rotation) of the board.

Multiple boards - it would be good to provide multiple separate boards and their
origins ... after all we are here for automation :)
