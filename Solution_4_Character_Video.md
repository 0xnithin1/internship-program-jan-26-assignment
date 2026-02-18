# Architecture: Character-Based Video Series Generator

> **Real-World Reference Implementation:**
> This architecture is based on **[LoraFrame (IDLock Engine)](https://github.com/Nithin9585/LoraFrame_)** — a project I built and presented at the **CENI AI Hackathon, Hyderabad** (Top 10 Finalist).
> LoraFrame is a persistent character memory & video generation system that combines episodic memory, LLM reasoning, and identity-preservation technology to create "permanent digital actors" that maintain visual consistency across generated images and videos.
> *(Note: This is my personal GitHub account — [Nithin9585](https://github.com/Nithin9585))*

## 1. Core Concept: "The Series Bible" (State Management)
To solve the hardest problem in AI video—**Consistency**—we introduce a centralized "Series Bible" database. This is not just metadata; it is a set of fine-tuned adapters and reference tensors that strictly enforce identity.

## 2. System Architecture

```mermaid
graph TD
    UserInput[User Prompt: 'Episode Idea'] --> ScriptAgent[Script Agent (LLM)]
    SeriesBible[(Series Bible DB)] --> ScriptAgent
    SeriesBible --> VisGen[Visual Generator]
    
    ScriptAgent -->|Screenplay JSON| Manager[Production Manager]
    
    subgraph "Visual Pipeline"
    Manager -->|Scene Desc| VisGen
    VisGen -->|IP-Adapter + LoRA| ImageGen[Stable Diffusion XL/Flux]
    ImageGen -->|Consistent Frames| VideoGen[Img2Video (Kling/SVD)]
    end
    
    subgraph "Audio Pipeline"
    Manager -->|Dialogue| TTS[ElevenLabs/OpenAI]
    TTS -->|Voice Profiles| AudioFiles
    end
    
    VideoGen --> Assembly[FFmpeg Assembly]
    AudioFiles --> Assembly
    Assembly --> FinalVideo
```

## 3. Component Details

### A. The "Series Bible" (Consistency Layer)
For every character (e.g., "Detective Mike"), we store:
1.  **Reference Images**: 5-10 clear shots (Front, Side, Close).
2.  **IP-Adapter Weights**: Pre-computed visual embeddings to inject into the diffusion model.
3.  **Voice ID**: A specific cloned voice handle (e.g., ElevenLabs ID).
4.  **Personality Prompt**: "Grumpy, cynical, speaks in short sentences."

### B. The Script Agent (LLM)
*   **Role**: Screenwriter.
*   **Input**: "Mike investigates a stolen pizza."
*   **Output**: A structured scene-by-scene script.
    *   *Scene 1*: "Mike's Office. Rainy." (Visual Spec)
    *   *Action*: "Mike looks at empty pizza box." (Animation Spec)
    *   *Dialogue*: "Mike: Who took the pepperoni?" (Audio Spec)

### C. The Visual Engine (Flux/SDXL + ControlNet)
*   **Problem**: Standard GenAI forgets what Mike looks like in Scene 2.
*   **Solution**:
    *   **Prompt**: "A photo of [Mike], looking angry at a pizza box..."
    *   **Conditioning**: Apply **IP-Adapter** (InstantID) using the Reference Images from the Bible. This forces the generated face to match Mike exactly, regardless of the prompt.
    *   **Motion**: Feed the generated frame into an Image-to-Video model (like Runway Gen-3 or Stable Video Diffusion) to generate 4 seconds of movement.

### D. The Audio Engine
*   Generate speech using the specific **Voice ID** from the Bible.
*   (Optional) Use **Wav2Lip** or **SadTalker** to sync the video lip movements to the generated audio.

### E. Assembly (FFmpeg)
*   Stitch all generated video clips ("shots").
*   Overlay audio tracks.
*   Add background music based on the "Mood" tag in the script.

## 4. Key Trade-off: Rendering Time vs. Quality
*   **Fast Mode**: Uses 2D static images with simple "camera pan" effects (Ken Burns). fast generation (mins).
*   **Full Video Mode**: Generates actual AI video for every shot. High compute cost, potential for "warping" artifacts, but premium output. We default to **Fast Mode** for draft, **Full Mode** for final export.
