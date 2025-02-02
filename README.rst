SolidPython
-----------

.. image:: https://circleci.com/gh/SolidCode/SolidPython.svg?style=shield
    :target: https://circleci.com/gh/SolidCode/SolidPython
.. image:: https://readthedocs.org/projects/solidpython/badge/?version=latest
    :target: http://solidpython.readthedocs.io/en/latest/?badge=latest
    :alt: Documentation Status

-  `SolidPython: OpenSCAD for
   Python <#solidpython--openscad-for-python>`__
-  `Advantages <#advantages>`__
-  `Installing SolidPython <#installing-solidpython>`__
-  `Using SolidPython <#using-solidpython>`__
-  `Importing OpenSCAD Code <#importing-openscad-code>`__
-  `Example Code <#example-code>`__
-  `Extra syntactic sugar <#extra-syntactic-sugar>`__

   -  `Basic operators <#basic-operators>`__
   -  `First-class Negative Space
      (Holes) <#first-class-negative-space-holes>`__
   -  `Animation <#animation>`__

-  `solid.utils <#solidutils>`__

   -  `Directions: (up, down, left, right, forward, back) for arranging
      things: <#directions-up-down-left-right-forward-back-for-arranging-things>`__
   -  `Arcs <#arcs>`__
   -  `Extrude Along Path <#extrude_along_path>`__
   -  `Basic color library <#basic-color-library>`__
   -  `Bill Of Materials <#bill-of-materials>`__

-  `solid.screw\_thread <#solidscrew_thread>`__
-  `Jupyter Renderer <#jupyter-renderer>`__
-  `Contact <#contact>`__
-  `License <#license>`__

SolidPython: OpenSCAD for Python
================================

SolidPython is a generalization of Phillip Tiefenbacher's openscad
module, found on
`Thingiverse <http://www.thingiverse.com/thing:1481>`__. It generates
valid OpenSCAD code from Python code with minimal overhead. Here's a
simple example:

This Python code:

::

    from solid import *
    d = difference()(
        cube(10),
        sphere(15)
    )
    print(scad_render(d))

Generates this OpenSCAD code:

::

    difference(){
        cube(10);
        sphere(15);
    }

That doesn't seem like such a savings, but the following SolidPython
code is a lot shorter (and I think clearer) than the SCAD code it compiles to:

::

    from solid import *
    from solid.utils import *
    d = cube(5) + right(5)(sphere(5)) - cylinder(r=2, h=6)

Generates this OpenSCAD code:

::

    difference(){
        union(){
            cube(5);
            translate( [5, 0,0]){
                sphere(5);
            }
        }
        cylinder(r=2, h=6);
    }

Advantages
==========

Because you're using Python, a lot of things are easy that would be hard
or impossible in pure OpenSCAD. Among these are:

-  built-in dictionary types
-  mutable, slice-able list and string types
-  recursion
-  external libraries (images! 3D geometry! web-scraping! ...)

Installing SolidPython
======================

-  Install via
   `PyPI <https://pypi.python.org/pypi/solidpython>`__:

   ::

       pip install solidpython

   (You may need to use ``sudo pip install solidpython``, depending on
   your environment. This is commonly discouraged though. You'll be happiest 
   working in a `virtual environnment <https://docs.python.org/3/tutorial/venv.html>`__ 
   where you can easily control dependencies for a given project)


Using SolidPython
=================

-  Include SolidPython at the top of your Python file:

   ::

        from solid import *
        from solid.utils import *  # Not required, but the utils module is useful

   (See `this issue <https://github.com/SolidCode/SolidPython/issues/114>`__ for 
   a discussion of other import styles

-  OpenSCAD uses curly-brace blocks ({}) to create its tree. SolidPython
   uses parentheses with comma-delimited lists. 
   
   **OpenSCAD:**

   ::

       difference(){
           cube(10);
           sphere(15);
       }

   **SolidPython:**

   ::

       d = difference()(
           cube(10),  # Note the comma between each element!
           sphere(15)
       )

-  Call ``scad_render(py_scad_obj)`` to generate SCAD code. This returns
   a string of valid OpenSCAD code.
-  *or*: call ``scad_render_to_file(py_scad_obj, filepath.scad)`` to store
   that code in a file.
-  If ``filepath.scad`` is open in the OpenSCAD IDE and Design => 'Automatic
   Reload and Compile' is checked in the OpenSCAD IDE, running
   ``scad_render_to_file()`` from Python will load the object in the
   IDE.
-  Alternately, you could call OpenSCAD's command line and render
   straight to STL.

Importing OpenSCAD code
=======================
- Use ``solid.import_scad(path)`` to import OpenSCAD code.

**Ex:** 

``scadfile.scad``
:: 

    module box(w,h,d){
        cube([w,h,d]);
    }

``your_file.py``
::

    from solid import *

    scadfile = import_scad('/path/to/scadfile.scad') 
    b = scadfile.box(2,4,6)
    scad_render_to_file(b, 'out_file.scad')

- Recursively import OpenSCAD code by calling ``import_scad()`` with a directory argument.

::

    from solid import *

    # MCAD is OpenSCAD's most common utility library: https://github.com/openscad/MCAD
    mcad = import_scad('/path/to/MCAD')

    # MCAD contains about 15 separate packages, each included as its own namespace
    print(dir(mcad)) # => ['bearing', 'bitmap', 'boxes', etc...]
    mount = mcad.motors.stepper_motor_mount(nema_standard=17)
    scad_render_to_file(mount, 'motor_mount_file.scad')

- OpenSCAD has the ``use()`` and ``include()`` statements for importing SCAD code, and SolidPython has them, too. They pollute the global namespace, though, and you may have better luck with ``import_scad()``,

**Ex:**
``scadfile.scad``
:: 

    module box(w,h,d){
        cube([w,h,d]);
    }

``your_file.py``
::

    from solid import *

    # use() puts the module `box()` into the global namespace
    use('/path/to/scadfile.scad') 
    b = box(2,4,6)
    scad_render_to_file(b, 'out_file.scad')


Example Code
============

The best way to learn how SolidPython works is to look at the included
example code. If you've installed SolidPython, the following line of
Python will print(the location of ) the examples directory:

::

    import os, solid; print(os.path.dirname(solid.__file__) + '/examples')
        

Or browse the example code on Github
`here <https://github.com/SolidCode/SolidPython/tree/master/solid/examples>`__

Adding your own code to the example file
`solid/examples/solidpython_template.py <https://github.com/SolidCode/SolidPython/blob/master/solid/examples/solidpython_template.py>`__
will make some of the setup easier.

Extra syntactic sugar
=====================

Basic operators
---------------

Following Elmo Mäntynen's suggestion, SCAD objects override the basic
operators + (union), - (difference), and \* (intersection). So

::

    c = cylinder(r=10, h=5) + cylinder(r=2, h=30)

is the same as:

::

    c = union()(
        cylinder(r=10, h=5),
        cylinder(r=2, h=30)
    )

Likewise:

::

    c = cylinder(r=10, h=5)
    c -= cylinder(r=2, h=30)

is the same as:

::

    c = difference()(
        cylinder(r=10, h=5),
        cylinder(r=2, h=30)
    )

First-class Negative Space (Holes)
----------------------------------

OpenSCAD requires you to be very careful with the order in which you add
or subtract objects. SolidPython's ``hole()`` function makes this
process easier.

Consider making a joint where two pipes come together. In OpenSCAD you
need to make two cylinders, union them, then make two smaller cylinders,
union them, then subtract the smaller from the larger.

Using hole(), you can make a pipe, specify that its center should remain
open, and then add two pipes together knowing that the central void area
will stay empty no matter what other objects are added to that
structure.

Example:

::

    outer = cylinder(r=pipe_od, h=seg_length)
    inner = cylinder(r=pipe_id, h=seg_length)
    pipe_a = outer - hole()(inner)

Once you've made something a hole, eventually you'll want to put
something, like a bolt, into it. To do this, we need to specify that
there's a given 'part' with a hole and that other parts may occupy the
space in that hole. This is done with the ``part()`` function.

See
`solid/examples/hole_example.py <https://github.com/SolidCode/SolidPython/blob/master/solid/examples/hole_example.py>`__
for the complete picture.

Animation
---------

OpenSCAD has a special variable, ``$t``, that can be used to animate
motion. SolidPython can do this, too, using the special function
``scad_render_animated_file()``.

See
`solid/examples/animation_example.py <https://github.com/SolidCode/SolidPython/blob/master/solid/examples/animation_example.py>`__
for more details.

solid.utils
===========

SolidPython includes a number of useful functions in
`solid/utils.py <https://github.com/SolidCode/SolidPython/blob/master/solid/utils.py>`__.
Currently these include:

Directions: (up, down, left, right, forward, back) for arranging things:
------------------------------------------------------------------------

::

    up(10)(
        cylinder()
    )

seems a lot clearer to me than:

::

    translate( [0,0,10])(
        cylinder()
    )

| I took this from someone's SCAD work and have lost track of the
  original author.
| My apologies.

Arcs
----

I've found this useful for fillets and rounds.

::

    arc(rad=10, start_degrees=90, end_degrees=210)

draws an arc of radius 10 counterclockwise from 90 to 210 degrees.

::

    arc_inverted(rad=10, start_degrees=0, end_degrees=90) 

draws the portion of a 10x10 square NOT in a 90 degree circle of radius
10. This is the shape you need to add to make fillets or remove to make
rounds.

Extrude Along Path
------------------

``solid.utils.extrude_along_path(shape_pts, path_pts, scale_factors=None)``

See
`solid/examples/path_extrude_example.py <https://github.com/SolidCode/SolidPython/blob/master/solid/examples/path_extrude_example.py>`__
for use.

Basic color library
-------------------

You can change an object's color by using the OpenSCAD
``color([rgba_array])`` function:

::

    transparent_blue = color([0,0,1, 0.5])(cube(10))  # Specify with RGB[A]
    red_obj = color(Red)(cube(10))                    # Or use predefined colors

These colors are pre-defined in solid.utils:

+------------+---------+--------------+
| Red        | Green   |  Blue        |
+------------+---------+--------------+
| Cyan       | Magenta |  Yellow      |
+------------+---------+--------------+
| Black      | White   |  Transparent |
+------------+---------+--------------+
| Oak        | Pine    |  Birch       |
+------------+---------+--------------+
| Iron       | Steel   |  Stainless   |
+------------+---------+--------------+
| Aluminum   | Brass   |  BlackPaint  |
+------------+---------+--------------+
| FiberBoard |         |              |
+------------+---------+--------------+

They're a conversion of the materials in the `MCAD OpenSCAD
library <https://github.com/openscad/MCAD>`__, as seen [here]
(https://github.com/openscad/MCAD/blob/master/materials.scad).

Bill Of Materials
-----------------

Put ``@bom_part()`` before any method that defines a part, then call
``bill_of_materials()`` after the program is run, and all parts will be
counted, priced and reported.

The example file
`solid/examples/bom_scad.py <https://github.com/SolidCode/SolidPython/blob/master/solid/examples/bom_scad.py>`__
illustrates this. Check it out.

solid.screw\_thread
-------------------

solid.screw\_thread includes a method, thread() that makes internal and
external screw threads.

See
`solid/examples/screw_thread_example.py <https://github.com/SolidCode/SolidPython/blob/master/solid/examples/screw_thread_example.py>`__
for more details.

Jupyter Renderer
----------------

Render SolidPython or OpenSCAD code in Jupyter notebooks using `ViewSCAD <https://github.com/nickc92/ViewSCAD>`__, or install directly via:
:: 

    pip install viewscad

(Take a look at the `repo page <https://github.com/nickc92/ViewSCAD>`__, though, since there's a tiny bit more installation required)

Contact
=======

Enjoy, and please send any questions or bug reports to me at
``evan_t_jones@mac.com``.

Cheers!

Evan

License
=======

This library is free software; you can redistribute it and/or modify it
under the terms of the GNU Lesser General Public License as published by
the Free Software Foundation; either version 2.1 of the License, or (at
your option) any later version.

This library is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser
General Public License for more details.

`Full text of the
license <http://www.gnu.org/licenses/old-licenses/lgpl-2.1.txt>`__.

Some class docstrings are derived from the `OpenSCAD User Manual
<https://en.wikibooks.org/wiki/OpenSCAD_User_Manual>`__, so 
are available under the `Creative Commons Attribution-ShareAlike License
<https://creativecommons.org/licenses/by-sa/3.0/>`__. 
