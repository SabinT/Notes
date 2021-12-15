# Links

## Good discussion about memory management when procedurally generating meshes

TLDR: `Mesh` objects aren't automatically GC'ed, because Unity holds on to them.

http://nothkedev.blogspot.com/2018/08/procedurally-generated-meshes-in-unity.html

## Mesh saver script to save mesh as asset

https://github.com/pharan/Unity-MeshSaver
MIT License

# Gotchas

## Don't use `OnValidate`

Instead use a 'dirty' flag, and use that to trigger updates.

https://medium.com/@ErikH2000/tips-for-procedural-generation-in-the-unity-editor-for-decorating-scenes-b714fd0daa9c
