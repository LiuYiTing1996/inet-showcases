Visualizing the Physical Environment
====================================

Goals
-----

The physical environment has a profound effect on the communication of
wireless devices. For example, physical objects like walls inside
buildings constraint mobility. They also obstruct radio signals often
resulting in packet loss. It's difficult to make sense of the simulation
without actually seeing where physical objects are.

The visualization of physical objects present in the physical
environment is essential. This showcase contains two example
simulations, which demonstrate the visualization of physical objects.

| INET version: ``4.0``
| Source files location: `inet/showcases/visualizer/environment <https://github.com/inet-framework/inet-showcases/tree/master/visualizer/environment>`__

About the visualizer
--------------------

The :ned:`PhysicalEnvironmentVisualizer` (also part of
:ned:`IntegratedVisualizer`) is responsible for displaying the physical
objects. The objects themselves are provided by the
:ned:`PhysicalEnvironment` module; their geometry, physical and visual
properties are defined in the XML configuration of the
:ned:`PhysicalEnvironment` module.

The two-dimensional projection of physical objects is determined by the
:ned:`SceneCanvasVisualizer` module. (This is because the projection is
also needed by other visualizers, for example, :ned:`MobilityVisualizer`.)
The default view is top view (z-axis), but you can also configure side
view (x and y axes), or isometric or orthographic projection.

The default view
----------------

This example configuration (``DefaultView`` in the ini file)
demonstrates the default visualization of objects. The objects are
defined in the ``indoor.xml`` file, which depicts an apartment with
three rooms. The network contains just two modules, a
:ned:`PhysicalEnvironment` and an :ned:`IntegratedVisualizer` module. When the
simulation is run, the network looks like this:

.. figure:: default.png
   :width: 80%
   :align: center

The isometric view
------------------

In this example configuration (``IsometricView`` in the ini file), the
view is set to isometric projection. This is done by setting the
:par:`viewAngle` parameter in :ned:`SceneVisualizer`:

.. literalinclude:: ../omnetpp.ini
   :start-at: viewAngle
   :end-at: viewAngle
   :language: ini

When the simulation is run, the network looks like the following.

.. figure:: isometric.png
   :width: 80%
   :align: center

3D visualization
----------------

The visualizer also supports OpenGL-based 3D rendering using the
OpenSceneGraph (OSG) library. If your OMNeT++ installation has been
compiled with OSG support, you can switch to 3D view using the toolbar.
The result will look like the following. Note that the
:ned:`SceneVisualizer` view settings have no effect on the 3D view, you can
use the mouse to move the camera and change the view angle.

.. figure:: 3d.png
   :width: 80%
   :align: center

Sources: :download:`omnetpp.ini <../omnetpp.ini>`, :download:`PhysicalEnvironmentVisualizationShowcase.ned <../PhysicalEnvironmentVisualizationShowcase.ned>`

Further information
-------------------

For more information, refer to the NED documentation of
:ned:`PhysicalEnvironmentVisualizer` and :ned:`SceneCanvasVisualizer`.

Discussion
----------

Use `this page <https://github.com/inet-framework/inet-showcases/issues/5>`__ in
the GitHub issue tracker for commenting on this showcase.
