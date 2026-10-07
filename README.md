# Costmap converter

`costmap_converter` is a ROS 2 plugin library for converting Nav2 costmaps into
polygonal obstacles. TEB loads the plugins directly through `pluginlib`; this
package does not provide a separate converter node.

The package targets Ubuntu 24.04 and ROS 2 Jazzy. Build it in the GRANDE
workspace with `colcon build --packages-up-to costmap_converter`.

## TEB configuration

Select a plugin and place its parameter overrides under the TEB controller
plugin's `costmap_converter` key. The TEB controller forwards this block to
the converter's private ROS 2 node. For example:

```yaml
controller_server:
  ros__parameters:
    FollowPath:
      costmap_converter_plugin: "costmap_converter::CostmapToPolygonsDBSMCCH"
      costmap_converter_spin_thread: true
      costmap_converter:
        cluster_max_distance: 0.4
        cluster_min_pts: 2
        cluster_max_pts: 30
        convex_hull_min_pt_separation: 0.1
```

The worker currently requires `costmap_converter_spin_thread: true`; TEB
rejects `false` during configuration because the private node has no external
executor. The dynamic-obstacle plugin receives the actual Nav2 costmap global
frame and uses the controller's ROS clock for its output timestamps.

Available plugins are listed in
[`costmap_converter/costmap_converter_plugins.xml`](costmap_converter/costmap_converter_plugins.xml).
The package retains the upstream BSD license and contributor attribution. The
multitarget tracker retains its separate GPLv3 license notices.
