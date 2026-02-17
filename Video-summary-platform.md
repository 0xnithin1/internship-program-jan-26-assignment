### **Video to Summary Platform: Problem Statement**

We have a local folder containing many long videos (often 3–4 hours each) and each file is large (200MB+). Watching these videos end-to-end to extract useful information is too time-consuming and does not scale.

We need an automated solution that processes every video in the folder and produces a concise “summary package” that can be consumed in 5–10 minutes. For each input video, the system must generate:

1. **Summary.md** (a structured Markdown note) containing:
   * Basic metadata (filename, duration, processed date)
   * A short high-level summary
   * Key highlights as bullet points with **timestamps**
   * Takeaways / action items
   * Links to extracted assets (clips and screenshots)
2. **Assets folder** containing:
   * **Highlight video clips** (short segments cut from the original video, aligned to the highlights)
   * **Screenshots** (important frames tied to highlights and timestamps)

The output must be organized in a predictable folder structure per video, for example:

```
input/videos/
  <video_name1>
  <video_name2>
output/
  <video_name1>/
  	Summary.md
	clips/
	screenshots/
  <video_name2>/
  	Summary.md
	clips/
	screenshots/

```

**Constraints**

* Must work for large files (200MB+) and long duration videos (3–4 hours).
* Must run in batch mode on a folder (no manual work per video).
* Highlights must include accurate timestamps and the generated clips/screenshots must match those timestamps.
* The final Markdown summary should be readable in 5–10 minutes and clearly link to the assets.

**Success Criteria**

Given a folder of videos, the system reliably produces one well-structured **Summary.md** plus relevant clips and screenshots for each video, allowing a user to understand the video’s core content quickly without watching the full recording.
