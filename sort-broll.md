---
name: broll-cutter-sorter
description: Split raw videos into individual B-roll subclips using scene detection, inspect frames visually, and sort clips into categorized folders.
---

# B-Roll Cutter & Sorter Workflow

When the user asks to cut, slice, or sort raw videos into B-roll folders, follow these exact steps inside the user's target directory. Never modify files outside the target directory, and never overwrite or delete original source videos.

## Step 1: Pre-Flight & Workspace Setup
1. Verify `ffmpeg` and `scenedetect` are available (`ffmpeg -version` and `scenedetect version`). If either is missing, tell the user what needs to be installed and stop.
2. Create a `_workspace` subfolder inside the target directory:
   - `_workspace/raw_cuts/` (for split subclips)
   - `_workspace/frames/` (for extracted preview thumbnails)
   - `_workspace/too_short/` (for micro-cuts under 1.5s)
   - `Sorted_Broll/` (for final categorized folders)
3. Find all video files (`.mp4`, `.mov`, `.mkv`, `.m4v`, `.MP4`, `.MOV`) in the root of the target folder. If there are more than 10 files, confirm the list with the user before processing.

## Step 2: Detect Scenes & Split Into Subclips
For each source video, run PySceneDetect with content detection and lossless stream-copy splitting:
```bash
scenedetect -i "<input_video>" detect-content -t 27 split-video -o "_workspace/raw_cuts" --copy
```
- Use `ffprobe` to check the duration of every generated subclip in `_workspace/raw_cuts/`.
- Move any subclip shorter than `1.5` seconds into `_workspace/too_short/` so accidental camera bumps or flash cuts do not clutter the library.

## Step 3: Extract Keyframes for Visual Inspection
For each valid subclip in `_workspace/raw_cuts/`:
- Extract a representative `640px`-wide thumbnail JPG (or 3 frames at 20%, 50%, and 80% timestamps for clips longer than 6 seconds) so you can inspect the shot without excessive token usage:
```bash
ffmpeg -y -i "<subclip_path>" -vf "thumbnail,scale=640:-1" -frames:v 1 "_workspace/frames/<subclip_name>.jpg"
```
## Step 4: Visually Classify & Propose Sorting Plan (Dry Run)
1. Inspect the extracted JPG frames in `_workspace/frames/` using your vision capability.
2. Group each subclip into clear, descriptive categories based on what is actually visible in the frame (for example: `Talking_Head_Aroll`, `Workspace_Tech`, `Product_Closeup`, `Outdoor_Establishing`, `Action_Movement`, `Unusable_Blurry`).
3. Present a markdown summary table showing:
   - **Proposed Folder Name**
   - **Clip Count**
   - **Brief Visual Summary & Sample Filenames**
4. **Stop and ask the user for confirmation** (allowing them to rename, merge, or adjust categories) before moving any subclips.

## Step 5: Sort & Clean Up
Once the user approves:
1. Create the approved category folders inside `Sorted_Broll/`.
2. Move and rename each subclip into its matching folder using `CATEGORY_###.<ext>` (preserving original file extensions).
3. Ask the user if they want to delete `_workspace/frames/` and `_workspace/too_short/` while keeping all original source videos untouched.
EOF
