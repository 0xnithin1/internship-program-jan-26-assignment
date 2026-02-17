### **Character-Based Short Video Series Generator (5-min Episodes): Problem Statement**

Users want to create a consistent short video series where each episode is around **5 minutes**. The series should reuse the same set of characters (with consistent look, personality, and relationships) across episodes, while allowing new situations and stories to be added episode-by-episode.

We need a system where the user can:

1. **Define a “Series Bible”**
   * Create characters with:
     * reference images (or generated character images)
     * personality traits, speaking style, and behavior rules
     * visual consistency notes (clothes, colors, age, expressions)
   * Define relationships between characters (friend, parent-child, rivals, mentor, etc.)
   * Optional: setting/world details (location, tone, recurring themes)
2. **Create an Episode**
   * User provides a short scene/story prompt (situation, conflict, outcome)
   * Select which characters appear in the episode (all or a subset)
   * Choose episode goal and style (comedy, motivational, slice-of-life, drama)
   * Set constraints: duration ~5 minutes, language, narration vs dialogues
3. **Generate the Output**
   The system generates a complete episode package:
   * Episode script (scene-by-scene with dialogues)
   * Shot list / storyboard plan (optional but preferred)
   * Visual assets needed per scene (character + background prompts or images)
   * Audio plan (voice lines per character, narration, background music cues)
   * Final rendered video aligned to the target duration

#### **Inputs**

* Series setup: characters (images + personality), relationships, world/style rules
* Episode prompt: situation + characters to include + desired tone + ending goal
* Output preferences: language, narration/dialogue ratio, platform format (9:16 / 16:9)

#### **Outputs**

* A 5-minute episode video (or a production-ready package to render it)
* Supporting assets: script + scene breakdown + prompts for visuals + voiceover lines

#### **Constraints**

* Character consistency across episodes (same face/style/voice/personality)
* Must respect relationships and behavioral rules
* Episode must fit ~5 minutes (control pacing and scene count)
* Should be easy to iterate: user edits story, regenerates scenes, or swaps characters
* Should support partial cast per episode (not always all characters)

#### **Success Criteria**

A user can define characters once, then repeatedly generate new 5-minute episodes by providing short story prompts. Each episode maintains consistent characters and relationships, produces coherent scenes and dialogues, and results in a video that matches the intended duration and style.
