PROJECT.md

MyCAD Geometry + FreeCAD Builder — System Architecture Overview

1. Core Concepts

Your system defines parametric 3D geometry using:

Shape — the main geometric object (walls, segments, panels)

Polygon — 2D footprint used for triangulation and side-face generation

Profile — 2D shape used for bosses/pockets

Features — Boss, Pocket, Hole

Faces — TOP, BOTTOM, FRONT, BACK, LEFT, RIGHT

2. Shape Geometry Model

A Shape contains:

bottom_points

top_points

polygon

features

transforms

_is_loft

loft_bottom

loft_top

Two geometry modes exist:

Extruded Mode

bottom_points and top_points are 3D

Top face is planar

Features on TOP face must use rotated axes

rotate3d() mutates geometry directly

polygon remains 2D and is not rotated

Lofted Mode

bottom_points and top_points are 2D XY

Top face is not planar

Lofted geometry is created by make_loft_solid

Features on TOP face use simple world axes

polygon is replaced by Polygon(bottom_pts)

_is_loft = True

3. Rotation Model

rotate3d() rotates:

bottom_points

top_points

loft_bottom

loft_top

It does not rotate:

polygon

feature positions

feature profiles

feature directions

Rotation is applied before building the solid. Transforms in shape.transforms (translate, rotate_z) are applied after feature operations.

4. Build Pipeline

Shape.build() calls:

make_solid(self)

make_solid:

Builds raw geometry

Applies features (holes, pockets, bosses)

Applies transforms

Returns final FreeCAD solid

5. Feature Placement Model

All profile-based features use:

_make_surface_profile_prism(shape, feature, direction, length)

This function determines the surface frame:

origin

x_axis

y_axis

z_axis (normal)

Features are transformed into world space using rotation, tilt, and face orientation.

6. Surface Frame Rules

Lofted Shapes (_is_loft = True)

Top face is not planar

Must use simple world axes

Extruded Shapes (_is_loft = False)

Top face is planar

Must compute rotated normal and tangent axes

7. Known Pain Points

Angled back wall (extruded)

Segment wall (lofted)

Mixed geometry (2D, 3D, FreeCAD.Vector)

8. Current Correct Approach

rotate3d() rotates geometry only

polygon stays 2D

Lofted shapes use simple axes

Extruded shapes compute normals

9. Next Steps

Feed modules into a new chat for full analysis and stabilization of the geometry pipeline.