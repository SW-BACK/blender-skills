# Agent Prompt Templates

详细的 Agent prompt 模板和 JSON Schema 定义。

---

## Requirements Analyst Schema

```json
{
  "type": "object",
  "properties": {
    "object_name": {
      "type": "string",
      "description": "The name of the object to create"
    },
    "object_type": {
      "type": "string",
      "enum": ["furniture", "industrial_equipment", "weapon", "vehicle", "building_element", "prop", "other"]
    },
    "use_case": {
      "type": "string",
      "enum": ["game", "digital-twin", "product-viz", "architectural", "general"]
    },
    "target_platform": {
      "type": "string",
      "enum": ["unity", "unreal", "threejs", "generic"]
    },
    "reference_urls": {
      "type": "array",
      "items": {"type": "string"},
      "description": "3-5 reference images found online"
    },
    "dimensions": {
      "type": "object",
      "properties": {
        "height": {"type": "string"},
        "width": {"type": "string"},
        "depth": {"type": "string"}
      }
    },
    "components": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": {"type": "string"},
          "description": {"type": "string"},
          "controllable": {"type": "boolean"},
          "states": {
            "type": "array",
            "items": {"type": "string"}
          },
          "animations": {
            "type": "array",
            "items": {"type": "string"}
          }
        },
        "required": ["name", "description", "controllable"]
      }
    },
    "animations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": {"type": "string"},
          "description": {"type": "string"},
          "duration_seconds": {"type": "number"},
          "affected_components": {
            "type": "array",
            "items": {"type": "string"}
          }
        },
        "required": ["name", "duration_seconds", "affected_components"]
      }
    },
    "api_requirements": {
      "type": "object",
      "properties": {
        "state_control": {
          "type": "array",
          "items": {"type": "string"}
        },
        "data_display": {
          "type": "array",
          "items": {"type": "string"}
        }
      }
    },
    "export_format": {
      "type": "string",
      "enum": ["fbx", "gltf", "obj"]
    },
    "polygon_budget": {
      "type": "string",
      "enum": ["low", "medium", "high"],
      "description": "low: <5k tris, medium: <15k tris, high: <50k tris"
    },
    "notes": {
      "type": "string"
    }
  },
  "required": ["object_name", "object_type", "components"]
}
```

---

## Technical Review Schema

```json
{
  "type": "object",
  "properties": {
    "status": {
      "type": "string",
      "enum": ["approved", "needs_revision"]
    },
    "issues": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "category": {
            "type": "string",
            "enum": ["hierarchy", "naming", "modularity", "animation", "api", "platform"]
          },
          "severity": {
            "type": "string",
            "enum": ["critical", "major", "minor"]
          },
          "description": {"type": "string"},
          "suggestion": {"type": "string"}
        },
        "required": ["category", "severity", "description"]
      }
    },
    "recommendations": {
      "type": "array",
      "items": {"type": "string"}
    },
    "revised_spec": {
      "type": "object",
      "description": "Updated specification if changes needed"
    }
  },
  "required": ["status"]
}
```

---

## Camera Position Code Templates

### Front View
```python
import bpy
cam = bpy.data.objects.get('Camera')
if cam:
    cam.location = (0, -7, 0)
    cam.rotation_euler = (1.57, 0, 0)
```

### Side View
```python
import bpy
cam = bpy.data.objects.get('Camera')
if cam:
    cam.location = (7, 0, 0)
    cam.rotation_euler = (1.57, 0, 1.57)
```

### Top View
```python
import bpy
cam = bpy.data.objects.get('Camera')
if cam:
    cam.location = (0, 0, 10)
    cam.rotation_euler = (0, 0, 0)
```

### Perspective View
```python
import bpy
cam = bpy.data.objects.get('Camera')
if cam:
    cam.location = (5, -5, 3)
    cam.rotation_euler = (1.1, 0, 0.785)
```

---

## Material State Template

```python
def create_state_materials(obj, state_config):
    """
    Create multiple materials for state switching.
    
    Args:
        obj: Blender object
        state_config: dict like {
            "idle": {"color": (0,1,0), "emission": 2.0},
            "active": {"color": (0,0,1), "emission": 3.0}
        }
    """
    for state_name, params in state_config.items():
        mat = bpy.data.materials.new(f"{obj.name}_{state_name}")
        mat.use_nodes = True
        nodes = mat.node_tree.nodes
        links = mat.node_tree.links
        
        # Clear default nodes
        nodes.clear()
        
        # Create emission shader
        emission = nodes.new('ShaderNodeEmission')
        emission.inputs['Color'].default_value = (*params['color'], 1)
        emission.inputs['Strength'].default_value = params.get('emission', 1.0)
        
        # Create output
        output = nodes.new('ShaderNodeOutputMaterial')
        
        # Connect
        links.new(emission.outputs['Emission'], output.inputs['Surface'])
        
        # Add to object
        obj.data.materials.append(mat)
    
    # Set first state as active
    if obj.data.materials:
        obj.active_material = obj.data.materials[0]
```

---

## Animation Template

```python
def create_simple_animation(obj, anim_name, start_loc, end_loc, duration_seconds, fps=24):
    """
    Create a simple location-based animation.
    
    Args:
        obj: Blender object
        anim_name: Name for the animation action
        start_loc: Starting location (x, y, z)
        end_loc: Ending location (x, y, z)
        duration_seconds: Animation duration
        fps: Frames per second (default 24)
    """
    import bpy
    
    # Calculate frame count
    frame_count = int(duration_seconds * fps)
    
    # Create action
    action = bpy.data.actions.new(name=f"{obj.name}_{anim_name}")
    
    # Assign action to object
    if not obj.animation_data:
        obj.animation_data_create()
    obj.animation_data.action = action
    
    # Set keyframes
    obj.location = start_loc
    obj.keyframe_insert(data_path="location", frame=0)
    
    obj.location = end_loc
    obj.keyframe_insert(data_path="location", frame=frame_count)
    
    # Set interpolation to smooth
    for fcurve in action.fcurves:
        for keyframe in fcurve.keyframe_points:
            keyframe.interpolation = 'BEZIER'
```

---

## Export Code Template

```python
import bpy
import os

def export_asset(object_name, output_dir, target_platform, export_animations=True):
    """
    Export 3D asset to appropriate format(s).
    
    Args:
        object_name: Name of the asset
        output_dir: Output directory path
        target_platform: 'unity', 'unreal', 'threejs', or 'generic'
        export_animations: Whether to bake animations
    """
    # Ensure output directory exists
    os.makedirs(output_dir, exist_ok=True)
    
    # Export FBX (Unity, Unreal)
    if target_platform in ['unity', 'unreal', 'generic']:
        fbx_path = os.path.join(output_dir, f"{object_name}.fbx")
        bpy.ops.export_scene.fbx(
            filepath=fbx_path,
            use_selection=False,
            object_types={'MESH', 'ARMATURE', 'EMPTY'},
            use_custom_props=True,
            bake_anim=export_animations,
            add_leaf_bones=False,
            path_mode='COPY',
            embed_textures=True
        )
        print(f"✅ Exported FBX: {fbx_path}")
    
    # Export glTF (Three.js, Web)
    if target_platform in ['threejs', 'generic']:
        gltf_path = os.path.join(output_dir, f"{object_name}.gltf")
        bpy.ops.export_scene.gltf(
            filepath=gltf_path,
            export_format='GLTF_SEPARATE',
            export_animations=export_animations,
            export_extras=True,
            export_apply=False
        )
        print(f"✅ Exported glTF: {gltf_path}")
    
    # Save Blender file
    blend_path = os.path.join(output_dir, f"{object_name}.blend")
    bpy.ops.wm.save_as_mainfile(filepath=blend_path)
    print(f"✅ Saved Blender file: {blend_path}")
```

---

## API Properties Template

```python
def add_api_properties(root_obj, spec):
    """
    Add custom properties for runtime API control.
    
    Args:
        root_obj: Root object of the asset
        spec: Specification dict with api_requirements
    """
    import bpy
    
    # Add state property if states defined
    components_with_states = [c for c in spec['components'] if c.get('states')]
    if components_with_states:
        root_obj["state"] = "default"
        root_obj.id_properties_ui("state").update(
            description="Current visual/functional state",
            default="default"
        )
    
    # Add data properties
    if 'data_display' in spec.get('api_requirements', {}):
        for prop_name in spec['api_requirements']['data_display']:
            root_obj[prop_name] = 0.0
            root_obj.id_properties_ui(prop_name).update(
                description=f"Runtime data: {prop_name}",
                min=0.0,
                max=100.0
            )
    
    # Add metadata
    root_obj["asset_version"] = "1.0"
    root_obj["generated_by"] = "blender-model-generator"
    root_obj["object_type"] = spec['object_type']
```

---

## Platform-Specific Integration Examples

### Unity C#

```csharp
using UnityEngine;

public class BlenderAssetController : MonoBehaviour
{
    private GameObject rootObject;
    private Dictionary<string, Material> stateMaterials;
    
    void Start()
    {
        // Find root object
        rootObject = GameObject.Find("AssetName_Root");
        
        // Cache state materials
        LoadStateMaterials();
    }
    
    public void SetState(string stateName)
    {
        // Find screen component
        var screen = rootObject.transform.Find("AssetName_Screen");
        if (screen != null && stateMaterials.ContainsKey(stateName))
        {
            screen.GetComponent<Renderer>().material = stateMaterials[stateName];
        }
    }
    
    public void PlayAnimation(string animName)
    {
        var animator = rootObject.GetComponent<Animator>();
        animator?.Play(animName);
    }
}
```

### Three.js JavaScript

```javascript
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

class BlenderAssetController {
    constructor() {
        this.loader = new GLTFLoader();
        this.model = null;
        this.mixer = null;
        this.animations = {};
    }
    
    async load(path) {
        return new Promise((resolve) => {
            this.loader.load(path, (gltf) => {
                this.model = gltf.scene;
                this.mixer = new THREE.AnimationMixer(this.model);
                
                // Cache animations
                gltf.animations.forEach(clip => {
                    this.animations[clip.name] = clip;
                });
                
                resolve(this.model);
            });
        });
    }
    
    setState(stateName) {
        const screen = this.model.getObjectByName('AssetName_Screen');
        if (screen) {
            // Switch material based on state
            const materialIndex = {
                'idle': 0,
                'active': 1,
                'error': 2
            }[stateName];
            
            if (materialIndex !== undefined) {
                screen.material = screen.material[materialIndex];
            }
        }
    }
    
    playAnimation(animName) {
        const clip = this.animations[animName];
        if (clip && this.mixer) {
            const action = this.mixer.clipAction(clip);
            action.reset().play();
        }
    }
    
    update(deltaTime) {
        if (this.mixer) {
            this.mixer.update(deltaTime);
        }
    }
}
```

---

这些模板可以在需要时引用，减少 SKILL.md 的长度。
