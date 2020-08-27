
============
KDL Overview
============

What can I use KDL for?
=======================

- **3D frame and vector transformations**: KDL includes excellent support to
  work with vectors, points, frame transformations, etc. You can calculate a
  vector product, transform a point into a different reference frame, or even
  change the reference point of a 6d twist. For more information take a look at
  the `geometry documentation <http://www.orocos.org/kdl/geometry>`_ on the KDL
  homepage.

- **Kinematics and Dynamics of kinematic chains**: You can represent a
  kinematic chain by a KDL Chain object, and use KDL solvers to compute anything
  from forward position kinematics, to inverse dynamics. For more information
  take a look at the `chain documentation <http://orocos.org/node/833>`_ on the
  KDL homepage. The `kdl_parser <https://wiki.ros.org/kdl_parser>`_
  includes support to construct a KDL chain from a XML Robot Description Format
  `(URDF) <https://wiki.ros.org/urdf/XML>`_ file.

- **Kinematics of kinematic trees**: You can represent a kinematic chain by
  a KDL Chain object, and use KDL solvers to compute forward position
  kinematics. Currently no other solvers are provided.
