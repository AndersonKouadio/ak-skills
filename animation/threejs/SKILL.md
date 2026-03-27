---
name: threejs
description: Three.js pour le web — scènes 3D, géométries, matériaux, lumières, shaders, textures, loaders, post-processing, interactions, animations. Déclenché pour tout travail 3D avec Three.js ou React Three Fiber.
---

# Three.js

Guide pour le développement 3D avec Three.js et React Three Fiber.

## Sujets couverts

| Sujet | Description |
|-------|-------------|
| Fondamentaux | Scène, caméra, renderer, boucle d'animation, resize |
| Géométrie | Formes primitives, BufferGeometry, vertices, faces |
| Matériaux | MeshBasicMaterial, MeshStandardMaterial, MeshPhysicalMaterial, ShaderMaterial |
| Lumières | AmbientLight, DirectionalLight, PointLight, SpotLight, shadows |
| Textures | TextureLoader, UV mapping, normal maps, environment maps |
| Shaders | GLSL vertex/fragment, uniforms, varyings, ShaderMaterial custom |
| Loaders | GLTFLoader, FBXLoader, OBJLoader, DRACOLoader, modèles 3D |
| Post-processing | EffectComposer, BloomPass, SSAOPass, UnrealBloomPass |
| Interactions | Raycaster, pointer events, drag, orbit controls |
| Animations | AnimationMixer, keyframes, morph targets, GSAP + Three.js |

## Avec React (React Three Fiber)

```tsx
import { Canvas } from "@react-three/fiber";
import { OrbitControls, Environment } from "@react-three/drei";

export default function Scene() {
  return (
    <Canvas camera={{ position: [0, 2, 5] }}>
      <ambientLight intensity={0.5} />
      <directionalLight position={[10, 10, 5]} />
      <mesh>
        <boxGeometry args={[1, 1, 1]} />
        <meshStandardMaterial color="orange" />
      </mesh>
      <OrbitControls />
      <Environment preset="studio" />
    </Canvas>
  );
}
```

## Packages clés

- `three` — Moteur 3D
- `@react-three/fiber` — React renderer pour Three.js
- `@react-three/drei` — Helpers (OrbitControls, Environment, Text3D, etc.)
- `@react-three/postprocessing` — Effets post-processing
- `leva` — GUI debug panel
- `@react-three/rapier` — Physique 3D
