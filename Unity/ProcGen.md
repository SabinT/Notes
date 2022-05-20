# APIs that might be useful for procedural/generative stuff
https://docs.unity3d.com/ScriptReference/Material.SetPass.html
https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html

# Making generated meshes readable
https://forum.unity.com/threads/making-a-custom-mesh-asset-readable-at-runtime.745658/
TLDR: make the mesh serializable as text, and edit the text file directly in VS Code. It will have a field called `m_IsReadable`

# Gotchas

## Good discussion about memory management when procedurally generating meshes

http://nothkedev.blogspot.com/2018/08/procedurally-generated-meshes-in-unity.html

## Mesh saver script to save mesh as asset

https://github.com/pharan/Unity-MeshSaver
MIT License

# Gotchas
TLDR: `Mesh` objects aren't automatically GC'ed, because Unity holds on to them.

## Don't use `OnValidate`

Instead use a 'dirty' flag, and use that to trigger updates.

https://medium.com/@ErikH2000/tips-for-procedural-generation-in-the-unity-editor-for-decorating-scenes-b714fd0daa9c
