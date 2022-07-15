# Feature Name: line_rendering_primitives

## Summary

Add primitives to make rendering lines easier

## Motivation

With the introduction of the `Material` and `AsBindGroup` abstraction it is now really easy to draw lines but it still requires knowledge of the feature and a bit of boilerplate. This RFC is intended to fix that.

## User-facing explanation

Rendering lines is useful for debugging purposes. It's already pretty easy to do when you know how `Material` and `Mesh` works, but it's hard to figure out if you aren't familiar with those or rendering in general.

Lines are defined in 2 ways

- `LineList`
  - Contains a list of lines where each line segment is a `start` and `end` point
  - A `Mesh` using `PrimitiveTopology::LineList`
- `LineStrip`
  - Contains a list of point and a line is drawn between each consecutive point
  - A `Mesh` using `PrimitiveTopology::LineStrip`

From a rendering perspective, a line is a combination of a `Mesh` and a `Material` specialized to use `PolygonMode::Line`.

To draw a line, you spawn it like any other `Mesh`, but instead of using a `StandardMaterial` you need to use a `LineMaterial`. The easiest way to spawn lines is by using a `LineBundle`.

If you want a wireframe, you can use a `LineMaterial` on a `Mesh` using `PrimitiveTopology::TriangleList`.

## Implementation strategy

- Create a new `line` module in `bevy_render`
- Create a `LineList` and `LineStrip` with a `impl From<Line> for Mesh`
- Create a `LineMaterial`
  - specializes with `PolygonMode::Line`
  - have a uniform `Color` to have colored lines
  - It only needs a really simple fragment shader to output the color
- Create a `LineBundle` that is similar to the `PbrBundle` but expects a `LineMaterial` instead of a `StandardMaterial`

## Drawbacks

I do not know of any at this time.

## Rationale and alternatives

The main goal is to make it more discoverable to beginners and reducing the boilerplate required to spawn lines without any third party plugins.

## Prior art

- PR implementing most of this RFC in an example: <https://github.com/bevyengine/bevy/pull/5319>
- bevy_polyline: <https://github.com/ForesightMiningSoftwareCorporation/bevy_polyline>

These examples are more of a showcase of what this api could eventually be used to build. The current proposal doesn't consider any kind of immediate mode api since it is out of scope, but the primitives could be used to build those apis.

- bevy_debug_lines: <https://github.com/Toqozz/bevy_debug_lines>
- Unity Debug.DrawLine: <https://docs.unity3d.com/ScriptReference/Debug.DrawLine.html>

## Unresolved questions

- Performance implications of using this approach. I do not know how to measure this. bevy_polyline mentions instanced rendering which is not really supported in this RFC. Since this is all based around high level `Mesh` and `Material` abstraction it could be updated to use instancing once those abstraction support it.
- If we want tesselated lines similar to what `lyon` can do should these be separate types or re-use the same types.
- These lines won't support any kind of thickness setting, is this acceptable?
- Should the wireframes use a `LineMaterial`?
  - It would be possible, but the wireframe material needs to override the `bias.slope_scale` which isn't required by the `LineMaterial`. Also it currently doesn't have a customizable color, but there's a PR to add it. See: <https://github.com/bevyengine/bevy/pull/5303>

## Future possibilities

The current proposal doesn't consider any kind of immediate mode api since it is out of scope, but the primitives could be used to build those apis. Lines are extremely useful when debugging.
