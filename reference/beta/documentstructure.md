## .v4p versus .vl
In vvvv beta a each patch has its own .v4p file. This is different with VL. Here many patches can be collected within a single .vl file or VL document, as we call it. Therefore small VL projects typically only have one VL document, even if they consist of multiple patches.

Then there are 3 different types of patches in VL:

* Datatype Patches
* The Document Patch
* Group Patches

## Datatype Patches
Datatype Patches are the ones that most closely correspond to what you're familiar with from vvvv beta. A new datatype patch is created by pressing `CTRL+P` or `CTRL+Shift+P`. Here you can patch as usual.

## The Document Patch
Every VL document has one toplevel patch that provides an overview of its content. You can always reach it by pressing `ALT+P`

## Group Patches
You can create a new Group patch by typing 'Group' in the Nodebrowser. Group patches are merely organisational elements in that hey have no purpose other than letting you structure/modularize your program.
