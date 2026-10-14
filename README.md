## Slope Analyzer

A Rhino/Grasshopper Python component that calculates slope percentage across a mesh and shows it as a vertex color gradient. Set a maximum slope, and the colors scale from flat to the limit so steep areas are easy to spot. It updates live as the terrain changes.


## Why this exists

Checking slope percentages by hand after every terrain change was slow and easy to get wrong. This tool replaces that check. Pair it with the [topography generator](README.md) and you can adjust the landform and see right away where it exceeds your slope limit.

## How it works

1. The input mesh is duplicated, so the original is never modified, and smooth vertex normals are computed.
2. For each vertex, slope is derived from the normal's Z component: `slope % = tan(θ) × 100`, where `cos(θ) = |normal.Z|`. Taking the absolute value means flipped normals don't matter.
3. Slope is divided by `max_slope` and clamped to 0–1, then mapped onto a color gradient.
4. Colors are interpolated linearly between the gradient stops and written to the mesh's vertex colors.

**Default gradient** (when no colors are supplied): green → yellow → red. Any slope at or above `max_slope` shows as the final color (red by default).

**Edge cases:**
- Near-vertical vertices (normal almost horizontal) are assigned a slope of 9999%, which maps to the final gradient color.
- Vertices with invalid or zero-length normals are colored gray.

## Inputs

| Name | Type | Description |
|---|---|---|
| `topo_mesh` | Mesh | Terrain mesh, e.g. from the topography generator |
| `path_mesh` | Mesh | Pathway mesh, analyzed separately (optional) |
| `max_slope` | Float | Slope percentage mapped to the end of the gradient (default `15.0`) |
| `gradient_colors` | List of colors | Gradient stops from flat to steepest. Optional; falls back to green/yellow/red |

## Outputs

| Name | Type | Description |
|---|---|---|
| `topo_slope` | Mesh | Copy of the terrain mesh with slope vertex colors |
| `path_slope` | Mesh | Copy of the path mesh with slope vertex colors |

Either mesh input can be left empty, and its output will be `None`.

## Setup

1. Add a **Python Script** component to your Grasshopper canvas.
2. Rename the inputs and outputs to match the tables above.
3. Paste in the contents of `slope_analyzer.py`.
4. Connect your meshes, set `max_slope`, and optionally connect a gradient or list of colors.

To flag areas steeper than a specific limit, set `max_slope` to that value. For example, `30` flags anything at or above 30% in the end color of the gradient.

**Requirements:** Rhino with Grasshopper (tested on: *[add Rhino version]*). No external libraries are needed beyond `Rhino.Geometry` and `System.Drawing`.

## Tips and limitations

- Slope is calculated at vertices from averaged normals, so it is a smoothed approximation, not an exact per-face value. A denser mesh gives a more accurate picture.
- The tool visualizes slope; it does not modify geometry or report a pass/fail count.
- The mesh must be oriented in world Z-up, since slope is measured against the world Z axis.
- Colors are shown through vertex colors, so make sure your Grasshopper preview or display mode shows them.

## Development notes

The first version of this script was drafted with AI coding assistants and then cross-checked with a second model. I directed the design and checked outputs against real site conditions. Planned improvements include unit tests for the slope and color-interpolation functions, and a summary of the percentage of area above the limit.

## License

MIT. See `LICENSE`.
