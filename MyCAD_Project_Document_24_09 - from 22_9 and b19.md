# MyCAD Project Document -regenerated from 22/09 document and modules in backup 19
 
#MyCAD Project Specification

A complete, unified, authoritative description of the MyCAD system — its architecture, geometry model, feature system, face‑placement engine, transform pipeline, FreeCAD backend, text subsystem, debugging tools, and public API.

#1. Overview

MyCAD is a parametric CAD geometry engine built around a simple core abstraction:

A Shape is an extruded polygon with a top surface, features, transforms, and face‑aligned geometry.

Everything else — bosses, pockets, holes, text, arrays, layout, boolean operations — is built on top of this foundation.

MyCAD integrates tightly with FreeCAD for solid generation, boolean operations, and text geometry.

#2. Core Geometry Model

2.1 Polygon

A Polygon is a 2D closed polyline lying strictly in the Z=0 plane.

Must have ≥ 3 points.

Points may be (x, y) or (x, y, z) with z == 0.

Supports:

Translation

Copying

Ear‑clipping triangulation

Convexity tests

Point‑in‑triangle tests

Edge enumeration

Offset (parallel polygon)

2.2 Profile

A Profile is one or more regions, each region:

[outer_polygon, hole_polygon, hole_polygon, ...]

Supports:

Multi‑region construction (Profile.from_regions)

2D rotation

3D rotation (Rodrigues’ formula)

2D/3D translation

Backend data storage (FreeCAD shape caching)

Copying (always strips Z → profiles remain 2D)

2.3 Shape

A Shape is an extruded polygon with:

bottom_points (copied from polygon)

top_points (initially identical)

features (bosses, pockets, holes)

transforms (rigid transforms)

top_z_value (uniform extrusion height)

Supports:

Uniform or per‑vertex top Z

Slant (shear top polygon)

Triangulated top surface interpolation

Top surface normals

Hole entry solver

Face coordinate systems

Face bounds

Face placement

Face arrays

Face layout engine

Boolean operations

Transform recording

Bounding box computation

#3. Feature System

3.1 Boss

Outward extrusion of a profile.

profile: Profile or Polygon

height: extrusion length

face: TOP/BOTTOM/FRONT/BACK/LEFT/RIGHT

pos: face‑local (u, v)

rotation, tilt_x, tilt_y

Backend surface frames

3.2 Pocket

Inward extrusion of a profile.

Same parameters as Boss

Depth instead of height

3.3 Hole

Cylindrical cut.

(x, y) or (x, y, z)

diameter

depth or through‑hole

direction (normalized 3D vector)

Blind hole entry solver

#4. Face Geometry System

4.1 Faces

Six faces:

TOP

BOTTOM

FRONT

BACK

LEFT

RIGHT

Each face defines a local coordinate system (u, v, w):

u = horizontal axis

v = vertical axis

w = outward normal

4.2 Face Bounds

Each face provides:

xmin, xmax, ymin, ymax, zmin, zmax

Used for:

placement

snapping

arrays

layout

4.3 Face Polygons

Each face returns a 4‑point polygon representing its boundary.

4.4 Face Placement

Functions:

place_profile_on_front_face

place_profile_on_back_face

place_profile_on_left_face

place_profile_on_right_face

place_profile_on_top_surface

place_profile_on_bottom_surface

4.5 Face Transforms

rotate_on_face

rotate_on_face_axis

mirror_on_face

scale_on_face

offset_on_face

4.6 Face Snapping

Snap profiles to:

center

edges

corners

midpoints

4.7 Face Arrays

Grid placement on any face.

4.8 Face Layout Engine

Full grid layout with:

rows, columns

margins

padding

alignment

#5. Transform System

Rigid transforms recorded on the Shape:

translate(dx, dy, dz)

rotate(angle, x, y, z) (clockwise‑positive)

Applied in order during build.

#6. Boolean System

BooleanShape(kind, a, b) supports:

cut

fuse

Build pipeline:

Build A

Build B

Apply boolean

Apply transforms

Flatten compound

#7. FreeCAD Backend

7.1 Shape → Solid

make_solid(shape):

Triangulate bottom

Triangulate top

Build side faces

Make shell → solid

Apply features

Apply transforms

7.2 Hole Cutter

Through holes: long cylinder

Blind holes: entry point solver → cylinder

7.3 Profile Prism

Vertical extrusion of profile regions. Supports backend‑cached FreeCAD shapes.

7.4 Surface‑Frame Prism

Unified boss/pocket builder for all faces.

Compute surface frame

Apply rotation + tilt

Transform profile points

Extrude along tilted normal

Cut holes

7.5 BuiltShape Wrapper

Provides text_on_face for FreeCAD solids.

#8. Text System

8.1 ShapeString → Profile

profile_text(text, height, font, tracking, deflection, x, y, angle):

Create FreeCAD ShapeString

Position at (x, y)

Rotate

Convert wires → polygons

Build multi‑region Profile

Cache backend FreeCAD shape

8.2 Integration

Text profiles can be:

bosses

pockets

boolean fused

boolean cut

placed on any face

arranged in arrays

used in layout engine

#9. Debugging Tools

9.1 show_polygon

Displays triangulated polygon in FreeCAD.

9.2 show_shape

Displays triangulated shape (bottom, top, sides).

#10. Public API

10.1 mycad/api

Exports:

Polygon

Shape

rectangle, circle, ellipse, centered_rectangle, slot

Hole

BooleanShape

Profile

FACE

text_profile

High‑level parts:

polygonal_box

rectangular_box

elliptical_box

lid

slide_box

10.2 mycad/init

Re‑exports the API.

#11. High‑Level Parts

11.1 Boxes

polygonal_box

rectangular_box

elliptical_box

11.2 Lids

Parametric lid generator.

11.3 Slide Box

Two‑part sliding enclosure.

#12. Build Pipeline Summary

Polygon → Shape → top/bottom geometry → features → transforms → FreeCAD solid

Features:

Profile → face placement → surface frame → extrusion → boolean

Text:

ShapeString → Profile → face placement → boss/pocket

Booleans:

Shape A → Shape B → build → cut/fuse → transforms

#13. Glossary

Polygon — 2D CCW polyline.

Profile — one or more regions of polygons.

Shape — extruded polygon with features.

Boss — outward extrusion.

Pocket — inward extrusion.

Hole — cylindrical cut.

Face — one of six surfaces of a shape.

Surface Frame — local coordinate system for face geometry.

Transform — rigid translation or rotation.

BooleanShape — lazy boolean node.

ShapeString — FreeCAD text geometry.

#14. Future Roadmap

Lofted shapes

Sweep profiles

Fillets/chamfers

Parametric joints

Constraint solver

SVG import

DXF export

Material system

#15. Conclusion

This document defines the complete MyCAD system — its geometry model, feature system, face engine, transform pipeline, FreeCAD backend, text subsystem, debugging tools, and public API.

It is now ready for distribution, onboarding, and future development.