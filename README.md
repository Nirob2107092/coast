# Dynamic Coastal Lighthouse Warning System

This project uses the OpenGL 3.3 programmable pipeline to draw a lighthouse,
an animated sea, and six editable vessels.

## Simple program flow

```text
Primitives.cpp calculates vertices and indices at runtime
                         |
                         v
Mesh::upload creates VAO + VBO + EBO
                         |
                         v
Vertex shader applies model, view and projection matrices
                         |
                         v
OpenGL assembles and rasterizes triangles
                         |
                         v
Fragment shader calculates material color and lighting
                         |
                         v
Framebuffer -> window
```

## What each file does

- `Main.cpp`: creates the window, owns the scene, processes input and draws one frame.
- `Primitives.cpp`: calculates box, cylinder, sphere, cone, grid, hull and sail geometry.
- `Renderer.h/.cpp`: owns each mesh's VAO, VBO and EBO and sends uniforms to the shaders.
- `Shader.h/.cpp`: compiles and links the vertex and fragment shaders.
- `Math3D.h`: calculates vectors and model/view/projection matrices.
- `Vessel.cpp`: combines primitive meshes to make three ships and three boats.
- `Lighthouse.cpp`: combines primitive meshes to make the island and lighthouse.
- `Sea.cpp`: recalculates wave heights and normals every frame.
- `Light.cpp`: provides the rotating spotlight's position and direction.
- `Camera.cpp`: calculates the camera position and view matrix.

## VAO, VBO and EBO

Every `Mesh` contains:

- `vertices`: CPU-side positions and normals.
- `indices`: CPU-side triangle or line indices.
- `vbo`: GPU buffer containing the vertices.
- `ebo`: GPU buffer containing the indices.
- `vao`: remembers the VBO/EBO bindings and vertex attribute layout.

Attribute location `0` is position and location `1` is normal. Drawing binds the
VAO and calls `glDrawElements`, so the EBO decides the vertex order.

## Runtime calculation (no precomputed model)

The project does not load an external or precomputed 3D model. Geometry is
calculated when the program starts:

- Circle angles are calculated with `2 * PI * segment / segmentCount`.
- Cylinder and cone vertices come from `sin` and `cos` of those angles.
- Sphere vertices come from latitude and longitude loops.
- Hull widths and heights are calculated from a normalized hull position.
- Lighthouse boulder positions and scales are generated using trigonometric formulas.
- Sail corner positions are calculated from width and height parameters.
- Sea vertices and normals are recalculated from wave equations every frame.
- Lighthouse beam direction is recalculated from elapsed time every frame.
- The spotlight is aimed down toward the sea, so boats and water brighten as
  the rotating beam passes over them.
- A shared `1.25` lighthouse root scale enlarges the complete island structure by 25% while
  preserving the proportions between the tower, house, rocks, stairs and pier.

Numbers such as a ship's position, material color, or lighthouse height are
design parameters, not precomputed mesh data.

## Scene objects

Ships:

1. Cargo ship: containers, bridge and crane.
2. Fishing trawler: cabin, net supports, winch and navigation light.
3. Naval patrol ship: narrow hull, bridge, radar mast and turrets.

Boats:

4. Rowboat: small hull, seats and two oars.
5. Sailing ship: wooden hull, two masts and generated sails.
6. Speedboat: pointed hull, engine, windshield and seats.

## Controls

- `1`-`6`: select a vessel.
- `T`, `R`, `S`: choose translation, rotation or scaling.
- Arrow keys and Page Up/Page Down: edit the selected vessel.
- `B`: toggle beacon edit mode.
- `L`: show or hide the visible beam.
- `M`: show or hide the moonlight.
- `H`: enable or disable lighthouse shadow mapping.
- `E`: enable or disable the planar water reflection map.
- Left mouse drag: orbit the camera.
- Mouse wheel: zoom.
- `P`: switch between first-person scene view and the overview view.
- `C`: reset the camera.
- `U`: aim the lighthouse beam upward.
- `D`: aim the lighthouse beam downward.
- `[` or `-`: reduce the lighthouse rotation speed.
- `]`, `+` or `=`: increase the lighthouse rotation speed.
- `Space`: pause or resume the rotating light at its current angle.
- `F11`: toggle fullscreen/windowed mode.
- `Esc`: quit.

The cargo ship and speedboat are animated automatically with smooth
time-based forward routes. Their positions are calculated from their yaw,
forward speed and elapsed time each frame, and the sea height is sampled at the
animated position, so they remain on the water while moving. The other four
vessels stay still and can all be edited with the controls above. The `U` and
`D` keys adjust the beam pitch while the lighthouse roof remains static; the
visible cone and the lighting direction use the same pitch.

The lighthouse renders a 1024 x 1024 depth shadow map before the main pass.
The fragment shader uses a 3 x 3 percentage-closer filter so vessels and the
island can cast soft shadows when the rotating beam reaches them. The sea also
has a planar reflection map: the opaque scene is rendered from a camera mirrored
across y=0, then the reflective water material samples that texture with a
Fresnel-style view-angle weight. `H` and `E` are provided for demonstrating the
two passes during the final presentation.

The sweep speed starts at 28 degrees per second. Speed changes are applied
without changing the current angle, and pausing freezes that angle until the
light is resumed.

Build `Debug|x64` or `Release|x64` from `coast.sln` in Visual Studio 2022.
