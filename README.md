## CLI Commands

Installation

```bash
npm i
```

Run dev mode

```bash
npm run dev
```

Build

```bash
npm run build
```

Run build

```bash
npm run preview
```

# About this project

**ModelDecay** introduces a solution for voxelization and animation within the [Three.js](https://threejs.org/) library. This tool enables developers to take a 3D model and convert it into a voxelized representation, subsequently animating the fall of these voxels. This project was created as a part of the "Extending the Three.js Framework" seminar given by the [Computer Graphics Systems](https://hpi.de/en/research/research-groups/computer-graphics-systems.html) chair at the [Hasso-Plattner-Institute](https://hpi.de/en/index.html). The prinicipal objective was to examine the possibility of vanishing/decaying 3D model as a type of object in Three.js tested specifically on said voxelization effect.   

## Usage
At first, the developer needs to create a Group containing the object(s) that should be voxelized. This group could for example be the loaded version of a model in a data format compatible with Three.js (the geometry also needs to be indexed). Afterwards, the developer can create a **ModelDecay** object and pass the necessary parameters.

Example:
```
//...initialization of the scene...//

// load the model 
const loader = new GLTFLoader();
let importedScene;
loader.load('models/Astronaut.glb';, (gltf) => {
  importedScene = gltf.scene;
});

// create the object
let decayingObject = new ModelDecay({
  model: importedScene, 
  resolution: 10,
  boxFill: 1,
  segments: 7,
  gravity: 3, 
  seed: 278947,
});

// add the model to the scene
scene.add(decayingObject);

// switch from the original model to the voxel representation (needed to play the animation)
decayingObject.switchToVoxelModel();

// to change back use
decayingObject.switchToOriginalModel();

//...continue initalization...//

function animate() {
  requestAnimationFrame(animate);
  
  // step animation 
  decayingObject.animate(//some time step; for instance delta time//);

  //reset the animation
  if (resetCondition) {
    decayingObject.reset();
  }
}
```
## Parameters

- **model**: the Group that should be voxelized
- **resolution**: determines the grid size along the largest dimension of the passed **model**
- **boxFill**: the relative size of one voxel with respect to the size of one grid cell
- **gravity**: the force with which the voxels will fall to the ground (the ground is determined by the relative transformation of the object)
- **segments**: the number of segments used for sampling the calculated animation path (7 seems to be enough according to empiric testing)
- **seed**: the seed used for randomizing the animation

## How it works
The implementation can be divided up in **Voxelization** and **Animation**.

### Voxelization
When googling how to voxelize meshes in Three.js, the search results present an approach using Raycasting. Raycasting, however, suffers from great performance issues which is why I looked for an alternative approach. This has led me to the paper [Computing Local Signed Distance Fields for Large Polygonal Models, by B. Chang et al]( https://doi.org/10.1111/j.1467-8659.2008.01210.x). This paper introduces a method for creating a voxelized representation of a mesh (i.e. its outer bounds) which I implemented. Afterwards, the voxels get painted by either the color of the mesh's material or by sampling a color from the mesh's texture based on the voxel's sorrounding vertices (vertex colors are not yet supported). The color of the inner voxels are determined by interpolating between those of the outer bound.

### Animation
The path a voxel traverses is calculated using [Catmull-Rom-Splines](https://en.wikipedia.org/wiki/Centripetal_Catmull%E2%80%93Rom_spline) which are already integrated in Three.js. The control points are the voxel's original position, a hypothetical contact point with another voxel, and a contact point with the floor. These points and the rotation are randomized using the **seed**. The resulting curve is then sampled corresponding to the **segments** parameter. The calculation of the speed is mainly based on the physics model of [Gravity on Inclined Planes](http://www.studyphysics.ca/newnotes/20/unit01_kinematicsdynamics/chp06_vectors/lesson25.htm) with some tweaks aiming to make the animation look more like a real physics simulation.

## Future Work
Even though the voxelization process is much faster than the Raycasting approach, the computational effort makes it unfit for real time applications. the voxelization and animation could be preprocessed, however, the resulting memory requirements would make it difficult to have multiple high resolution voxelized representations which have to be loaded in one scene at the same time. In order to achieve this, the algorithm would have to be ported to the GPU. This should be possible since most steps are trivially parallelizable, but such a solution has yet to be implemented.

## Attributions
The models used in this demo:

- [Astronaut](https://poly.pizza/m/dLHpzNdygsg) by [Poly by Google](https://poly.pizza/u/Poly%20by%20Google) \[CC-BY\] via Poly Pizza
- [Earth](https://skfb.ly/oPZWW) by RamonaFlowers🌼[https://sketchfab.com/ramonaflowers] is licensed under [Creative Commons Attribution](http://creativecommons.org/licenses/by/4.0/)
- [Chess Set](https://sketchfab.com/3d-models/chess-set-2ed9658a98df4dfc91d3a4048c4ea3a9) by [Marcin Urbański](https://sketchfab.com/Marcin_Urbanski) is licensed under [Creative Commons Attribution](http://creativecommons.org/licenses/by/4.0/)
