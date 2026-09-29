# ComfyUI Setup Guide for 3D Character UV Texturing from Human Photos

This guide explains how to set up ComfyUI to texture UV maps of 3D character models using images of real human persons, based on the workflow described in your web search results.

## ✅ Installation Status

The following components have been successfully installed/cloned:

1. **ComfyUI-3D-Pack** - ✅ Already installed
   - For loading/exporting OBJ/FBX/GLB and baking to UV space

2. **ComfyUI ControlNet Auxiliary Preprocessors** - ✅ Installed via CLI
   - For generating normal maps, depth maps, etc. from 3D renders

3. **ComfyUI IPAdapter Plus** - 🔄 Cloned directly
   - For feeding in reference photos of the person to condition the sampler

4. **InstantID** - 🔄 Cloned directly + Dependencies installed
   - For strong facial identity transfer from reference photos

## 📋 Manual Installation Steps (if needed)

If you need to install any components manually, here are the exact steps:

### 1. Install via ComfyUI Manager CLI (Preferred)
```bash
# From your ComfyUI directory:
python custom_nodes/ComfyUI-Manager/cm-cli.py install comfyui_controlnet_aux
python custom_nodes/ComfyUI-Manager/cm-cli.py install ipadapter
python custom_nodes/ComfyUI-Manager/cm-cli.py install instantid
```

### 2. Direct Git Clone (Alternative)
```bash
# IPAdapter Plus
git clone https://github.com/cubiq/ComfyUI_IPAdapter_plus custom_nodes/ComfyUI_IPAdapter_plus

# InstantID  
git clone https://github.com/cubiq/ComfyUI_InstantID custom_nodes/ComfyUI_InstantID

# Install InstantID dependencies
pip install -r custom_nodes/ComfyUI_InstantID/requirements.txt
```

### 3. Required Base Components
Make sure you also have:
- ComfyUI Manager (already present in your setup)
- A realistic-skin-capable checkpoint (SD1.5, SDXL, or Flux with appropriate support)
- ControlNet models (ControlNet-Normal for position/normal maps, ControlNet-Tile for UV-space painting)

## 📥 Downloading Photorealistic Checkpoints

For UV texturing of human characters, you'll need a photorealistic checkpoint. Here are recommended options:

### SD1.5 Checkpoints (Good for detailed skin textures):
- **Realistic Vision V6.0**: https://huggingface.co/SG161222/Realistic_Vision_V6.0_B1noVAE
  - Download: `Realistic_Vision_V6.0_B1noVAE.safetensors`
  - Note: This version doesn't include VAE - you'll need to use the default VAE or download one separately
- **Realistic Vision V5.1**: https://huggingface.co/SG161222/Realistic_Vision_V5.1
  - Includes VAE in the checkpoint

### SDXL Checkpoints (Higher quality, better for 4K textures):
- **Juggernaut XL**: https://huggingface.co/rundiffusion/Juggernaut-XL-v9
  - Excellent for photorealistic skin textures
  - Version v9 or later recommended
- **RealVisXL V4.0**: https://huggingface.co/SG161222/RealVisXL_V4.0
  - Another excellent photorealistic option
- **EpicRealism XL**: https://huggingface.co/emilianJR/epiCRealismXL

### How to Install:
1. Download the `.safetensors` or `.ckpt` file from the links above
2. Place it in: `D:\UE\Tailwind_R E B U I L D\Resources\Code\Refs\Repos\LLM\ComfyUI\models\checkpoints\`
3. Restart ComfyUI to see the new checkpoint in the loader dropdown

### Recommended VAEs (if needed):
Some checkpoints require separate VAE files:
- **sd-vae-ft-mse**: https://huggingface.co/stabilityai/sd-vae-ft-mse-original
- Place in: `models\vae\`

### ControlNet Models (Required for workflow):
Place these in `models\controlnet\`:
- **ControlNet-Normal**: For normal/position map conditioning
  - https://huggingface.co/lllyasviel/ControlNet-v1-1/tree/main (look for control_v11p_sd15_normalbae)
  - Or XL version if using SDXL checkpoint
- **ControlNet-Tile**: For UV-space painting structure preservation
  - Same location as above (control_v11f1e_sd15_tile)

## 🔧 Workflow Setup

Based on your web search results, here's how to configure the workflow:

### Approach: UV-space Generation (Recommended for character meshes)

1. **Prepare UV Maps** (in your DCC tool like Blender, Substance, Marmoset):
   - Bake character's normal map, position map (world/tangent-space XYZ as RGB)
   - Bake AO/curvature maps into UV layout
   - These maps will be fed into ControlNet for structure preservation

2. **ComfyUI Node Setup**:
   - Load your base checkpoint (photorealistic SD/SDXL/Flux model)
   - Load ControlNet models:
     * ControlNet-Normal (for position/normal map conditioning)
     * ControlNet-Tile (for UV-space painting structure preservation)
   - Load IPAdapter Plus nodes for reference photo conditioning
   - Load InstantID/ReActor nodes for facial identity preservation
   - Setup KSampler with appropriate settings

3. **Reference Photo Processing**:
   - Load 2-4 varied-angle photos of the person
   - Pass through IPAdapter (or IPAdapter FaceID Plus) for texture/feature conditioning
   - Pass through InstantID for strong facial identity preservation
   - Combine these conditionings in the sampler

4. **Sampling Settings**:
   - Prompt: "seamless character skin texture, photographic detail, even lighting, no shadows"
   - Keep denoise moderate (0.5-0.7) if starting from base texture
   - Use even, shadow-free lighting prompt (crucial for texture baking)

5. **Post-processing**:
   - Fix seams and unseen regions (under arms, back of ears, etc.)
   - Use inpaint mask pass on seam pixels or texture-dilation/padding node
   - Export final texture as PNG/EXR at target resolution (2K/4K)

### Alternative: Multi-view Projection
For more complex geometry with occlusion:
1. Render depth/normal passes from multiple camera angles around mesh
2. Generate matching images per view with ControlNet+IPAdapter
3. Project each view back onto mesh and bake to texture atlas
4. Requires projection-bake step in Blender or 3D-Pack nodes

## 🎮 Integration with Unreal Engine

1. Save final texture as PNG/EXR at 2K/4K resolution
2. If generating full material set, run through normal/roughness split
3. Drop into UE5 character's material instance alongside existing UV-mapped mesh
4. Ensure UV mapping matches between your 3D model and the generated texture

## 📝 Notes & Best Practices

- **Lighting Consistency**: Use "even lighting, no shadows" in prompts to avoid baking shadows that will look wrong when relit in-engine
- **Photo Reference**: Use 2-4 varied-angle photos showing different facial expressions and lighting
- **Seam Handling**: Pay special attention to UV island boundaries and occluded areas
- **Testing**: Start with lower resolution tests before committing to 4K textures
- **Model Selection**: Photorealistic checkpoints work best for human skin textures
- **Checkpoint Compatibility**: Ensure your ControlNet models match your checkpoint version (SD1.5 vs SDXL)

## 🗂️ File Locations

- Custom nodes installed in: `custom_nodes/`
- IPAdapter Plus: `custom_nodes/ComfyUI_IPAdapter_plus/`
- InstantID: `custom_nodes/ComfyUI_InstantID/`
- ControlNet Aux: `custom_nodes/comfyui_controlnet_aux/`
- 3D Pack: `custom_nodes/ComfyUI-3D-Pack/`
- Checkpoints: `models/checkpoints/`
- ControlNet models: `models/controlnet/`
- VAEs: `models/vae/`

## 🚀 Next Steps

1. Restart ComfyUI to recognize all newly installed nodes
2. Download appropriate checkpoint and ControlNet models (see section above)
3. Prepare your 3D character's UV maps and normal/position maps
4. Gather reference photos of the human subject
5. Build and test the workflow as outlined above

---

*Setup completed on: 2026-09-27*
*Based on workflow research from web search results*