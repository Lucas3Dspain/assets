
# How to take advantage of subcomponents in production

<img src="screenshots/Bucket_articulated.gif" width="400"/>

## What are Subcomponents?
In USD terms, [subcomponents](https://openusd.org/release/glossary.html#usdglossary-subcomponent)
are a [kind](https://openusd.org/release/glossary.html#usdglossary-kind) that can be assigned to a prim.
[see more](https://github.com/usd-wg/assets/blob/main/docs/asset-structure-guidelines.md).

## Composition

todo: solve internal note, refine example compositions


internal note: There are many ways to structure subcomponents.\ 
I would like to share it with the group and study pros and cons with the hope to come up with a go-to structure.

Main topics:
- Should subcomponent prim replicate component structure geo, mtl scopes.\
It looks counterintuitive to have geo outside /geo?\
pd: looking at assemblies it works that way, so it might not be an issue. Is it different here?\
we don't bring any asset from outside the component as component models should be Self-contained?
- Is there any issue having meshes have kind subcomponent directly applied? or is better to only apply it to xforms? 


**Simple Subcomponent** (Bucket example) Variations to consider:

```
/Bucket                 (xform) ← Component Kind
    /geo                (scope)
        /bucket         (mesh)
        /handle         (mesh) ← Subcomponent Kind★
    /mtl                (scope)
/\_\_class\_\_          (class)
    /Bucket             (class) ← Inherited by /<component>
```

```
/Bucket                 (xform) ← Component Kind
    /geo                (scope)
        /bucket         (mesh)
        /handle         (xform) ← Subcomponent Kind★
            /wire       (mesh)
            /handle     (mesh)
    /mtl                (scope)
/\_\_class\_\_          (class)
    /Bucket             (class) ← Inherited by /<component>
```

Subcomponents should be outside of geo, right?

```
/Bucket                 (xform) ← Component Kind
    /geo                (scope)
    /mtl                (scope)
    /handle             (xform) ← Subcomponent Kind★
        /geo            (scope)
/\_\_class\_\_          (class)
    /Bucket             (class) ← Inherited by /<component>
```

**Simple Subcomponent - With Purpose Scopes** (Bucket example)

```
/Bucket                 (xform) ← Component Kind
    /geo                (scope)
        /proxy          (scope) ← Purpose: proxy★
        /render         (scope) ← Purpose: render★
    /mtl                (scope)
    /handle             (scope) ← Subcomponent Kind★
        /geo            (scope)
            /proxy      (scope) ← Purpose: proxy★
            /render     (scope) ← Purpose: render★
        /mtl            (scope)
/\_\_class\_\_          (class)
    /Bucket             (class) ← Inherited by /<component>
```

**Shallow Subcomponent** (WoodenTable example)

```
/WoodenTable            (xform) ← Component Kind
    /geo                (scope)
        /render         (scope)
        /proxy          (scope)
        /door           (scope) ← Subcomponent Kind★
            /render     (scope)
            /proxy      (scope)
        /drawers        (xform) ← Group Kind★
            /drawer1    (scope) ← Subcomponent Kind★
                /render (scope)
                /proxy  (scope)
            /drawer2    (scope) ← Subcomponent Kind★
            /drawer3    (scope) ← Subcomponent Kind★
            /drawer4    (scope) ← Subcomponent Kind★
    /mtl                (scope)
/\_\_class\_\_          (class)
    /WoodenTable        (class) ← Inherited by /<component>
```

Subcomponents should be outside of geo, right?

```
/WoodenTable        (xform) ← Component Kind
    /geo            (scope)
    /mtl            (scope)
    /door           (scope) ← Subcomponent Kind★
        /geo        (scope)
        /mtl        (scope)
    /drawers        (xform) ← Group Kind★
        /drawer1    (scope) ← Subcomponent Kind★
            /geo    (scope)
            /mtl    (scope)
        /drawer2    (scope) ← Subcomponent Kind★
        /drawer3    (scope) ← Subcomponent Kind★
        /drawer4    (scope) ← Subcomponent Kind★
/\_\_class\_\_      (class)
    /WoodenTable    (class) ← Inherited by /<component>
```

**Nested Subcomponent** (Antenna example)

```
/Antenna                (xform) ← Component Kind
    /geo                (scope)
    /mtl                (scope)
    /base               (scope) ← Subcomponent Kind★
        /geo            (scope)
        /mtl            (scope)
        /upperArm       (scope) ← Subcomponent Kind★
            /geo        (scope)
            /mtl        (scope)
            /lowerArm   (scope) ← Subcomponent Kind★
                /geo    (scope)
                /mtl    (scope)
                /...
/\_\_class\_\_      (class)
    /Antenna        (class) ← Inherited by /<component>
```

## Uses for Subcomponents

### Proxy mesh generation

To generate the proxy purpose geometry, we can make it manually if needed or, more commonly,
using an automated process that merges the meshes into one prim and optimizes the topology. This is something very common, what we did was use the
kind:subcomponent to filter prims so that we don't merge them, that way we can still have articulation represented in the proxy purpose.

The geometry under the proxy scopes should have:
- as few prims as possible without loosing articulated pieces.
- as few polygons while maintain volume and silhouette.

### Articulations

<img src="screenshots/articulated_assets_example.gif" alt="Example Assets" width="500"/>\

Articulations are movable elements within the asset.
It typically involves child primitives with strategically placed pivots and is identified by a kind:subcomponent designation.
These articulations serve as the adjustable elements that contribute to the asset's flexibility and adaptability.

Articulations are ***rigid*** when only xform transformations are required.

Articulations can be adjusted to reduce asset repetition and to adapt to their surroundings, creating a more cohesive look in the scene.

note: this workflow allows maximum flexibility at the cost of instancing. see more in the Optimization & instancing section.

todo: insert gif showing dressing (transform + pose)

#### Optimization & instancing
Articulated assets come with a limitation – they cannot be both posed and instanced simultaneously.\
This is due to the fact that when a parent prim is made instanceable, the child prims won't receive any edits.

To address this limitation, a workaround can be employed, particularly when a few distinct poses are enough.

One effective workaround involves creating a "pose" VariantSet within the Model's defaultPrim,
acting as a repository for a library of poses. Variants are added to this VariantSet,
offering a flexible solution for managing different poses that can be instanced.

ignore note: is there a way to make the subcomponents instanceable so only a xform is live. but the mesh is instanced in any range of movement ?

todo: Add link to gif showing asset variants in usdview

todo: Add link to example_dressing_variants.usda

Additionally, extra poses can be added from a Dressing/Shot definition. see more in the example file.

todo: Add link to example_dressing_variants.usda


### Autorig & rig swapping

Not done yet. Coming soon












## Objective
In this page we are going to describe how subcomponents can be used in a vfx/animation production for different workflows.

asset definition
- definition
- pivot
- lod generation
- autorig

set dressing
- use of subcomponent prim to pose the asset to reduce repetition and improve set adaptation.
- optimizing poses with variants while reducing repetition

animation
- swap for rig and back (rigid vs deformable articulation round tripping)

Show ways to laverage usd composition and subcomponents to describe efficient articulated assets.
 - what are subcomponents? (no need)
 - What are articulated assets?
 - Benefits and challenges
 - Composition examples
 - laverage examples
 - limitations


## What are Subcomponents and Articulated Assets?

<img src="screenshots/articulated_assets_example.gif" alt="Example Assets" width="500"/>\

In USD terms, [subcomponents](https://openusd.org/release/glossary.html#usdglossary-subcomponent) are a [kind](https://openusd.org/release/glossary.html#usdglossary-kind) that can be assigned to a prim. [see more](https://github.com/usd-wg/assets/blob/main/docs/asset-structure-guidelines.md).

Subcomponents allow easy selection of the articulated pieces.\
DCCs have different behaviors, but usually they allow the user to pick a selection-mode that will 
make selections based on the prim's kind.

For this guide, we will say that an asset is *Articulated* if it has any articulated pieces.\
Articulated pieces are child prims with a well-placed pivot and kind:subcomponent.

For instance, consider an Oven asset; rotating the door (articulated piece) achieves the open/closed pose.


### What are articulated assets?


Like in many other usd definitions, assets/models can be described as detailed as needed wanted. ex: you might have a building as a single component asset with one prim. Or have a very detailed assembly model that uses other assets like flowerpots, windows shutters...etc with that said, articulation is an extra level of detail/complexity and is not needed for every asset.

So what do I mean when I say articulated? Articulation can be __*rigid*__ , when only an xform transformation is required or __*deformable*__ when the points/vertex need to be transformed non-uniformly. At the end of the day rigid articulations can be described as a deformable ones but that is inefficient.

When using subcomponents and for this page, we are gonna only make use of static rigid articulations.

### Bennefits and challenges
Benefits

Challenges

Articulated assets can't be instanceable, at least not if we want to have an edit on a child prim. There is a way to optimize this by generating variant at the rootprim, making a "library" of articulation posibilities

### Composition examples
- simple rigid 
### laverage examples
- generate lods for articulable assets
- autorig articulable assets

### Limitations
![Corck screw openner](screenshots/CorckScrewOpenner.png)

Assets where mechanical pieces need to work together, cant have that behaviour described at the current state of openUsd.\
It will need a 3rd party rig like a maya rig.

## Credits

-- Model Credits --

https://sapien.ucsd.edu/browse

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) by the Academy Software Foundation.
