# Color Measurement and Correction Project
## Step-by-step project plan and technical outline

**Project type:** Python, computer vision, color measurement, and data-driven color correction  
**Initial setup:** Smartphone camera + ivory/white A4 paper + JAMK printer (printing available from Monday)  
**Long-term goal:** Explore whether a camera-based workflow can measure color variation and recommend adjustments that move a new sample closer to a target color.

---

## 1. Project idea in plain language

In ceramic tile printing, a standard color may look different in a later production run. The original workflow used CMYK printing and Photoshop Levels adjustments, followed by firing. The project will first build a small, controlled prototype on paper:

1. Keep a **target reference**: the desired color originally associated with ivory paper.
2. Measure the paper/background color separately.
3. Photograph a known color sample with a smartphone.
4. estimate its color values from the image.
5. Compare the measured sample with the target.
6. Test controlled color changes and record which changes move the sample closer to the target.
7. Only after collecting enough reliable experiments, explore a model that recommends a correction.

The paper prototype is a learning and proof-of-concept stage. Paper, ink, lighting, camera processing, and fired ceramic surfaces behave differently. Success on paper does **not** by itself prove that the same correction will work on fired tiles.

---

## 2. What the first version will and will not do

### First version will
- Read a photo from a phone.
- Let us identify the target/sample patches in the photo.
- Estimate average color values from a selected patch.
- Record measurements and metadata in a CSV file.
- Compare a sample against a reference using CIELAB and Delta E, where calibration quality permits.
- Show the direction and size of the measured difference.
- Keep a repeatable record of each experiment.

### First version will not
- Claim laboratory-grade color measurement from an uncalibrated phone photo.
- Automatically know the exact CMYK correction from one photo.
- Treat ivory and white paper as interchangeable.
- Claim paper results predict post-firing ceramic color without real tile data.
- Train a meaningful AI/ML model before there are enough controlled input/output examples.

---

## 3. Workflow overview

```text
Define the target and experiment
          |
          v
Photograph target paper and target color
          |
          v
Record paper/background color separately
          |
          v
Set up repeatable phone-photo conditions
          |
          v
Before printing: test image reading and patch sampling
          |
          v
Monday: generate and print a labeled color chart on A4
          |
          v
Photograph the chart under controlled conditions
          |
          v
Detect/select patches and calculate color values
          |
          v
Compare with target (CIELAB / Delta E)
          |
          v
Run controlled correction experiments
          |
          v
Build a dataset and evaluate correction rules/model
          |
          v
Prototype report/dashboard; discuss ceramic validation
```

---

## 4. Step-by-step plan

### Phase 0 — Define the target and keep evidence
**When:** Before coding or printing

1. Choose one target color to start with. Do not begin with dozens of colors.
2. Keep the physical ivory paper and the standard color sample if available.
3. Take separate photos of:
   - the blank ivory paper,
   - the standard color patch on ivory paper,
   - the blank white paper that will be used for the experiment.
4. Give each photo a clear filename, for example:
   - `reference_ivory_blank_01.jpg`
   - `target_color_on_ivory_01.jpg`
   - `test_white_blank_01.jpg`
5. Record the phone model if known, date/time, lighting, flash setting, and paper type.
6. Do not use a screenshot or an image sent through a service that may alter its colors if the original photo is available.

**Important:** A photo of ivory paper is a camera-based estimate, not a certified measurement. The camera may change exposure, white balance, contrast, and saturation automatically.

**Deliverable:** Reference photos plus a short notes file.

### Phase 1 — Set up repeatable photography
**When:** Before and after printing

Use the same setup each time as far as possible:

- Place the page flat.
- Hold the phone directly above it, with the lens parallel to the page.
- Use diffuse, even light; avoid mixed light sources, glare, and hard shadows.
- Turn flash off.
- Avoid filters, portrait mode, night mode, and automatic “enhancement” if these can be disabled.
- Keep distance and framing consistent.
- Include the whole page and its edges so we can check perspective.
- Take at least three photos of the same sample to estimate repeatability.

If possible, lock focus, exposure, and white balance. If the phone cannot lock them, record this limitation and keep the setup as consistent as possible.

**Deliverable:** A repeatable photo procedure and multiple test images.

### Phase 2 — Before printing: learn to read an image in Python
**When:** Before Monday

We can start without a printed chart.

1. Install or verify Python and the required libraries:
   - `opencv-python` for image loading and pixel operations,
   - `numpy` for numerical calculations,
   - `pandas` for measurement tables,
   - `matplotlib` for optional plots.
2. Load one reference photo with OpenCV.
3. Check that the image opened correctly and inspect its pixel dimensions.
4. Select a small region inside the blank paper and another region inside the target color patch.
5. Calculate the mean and spread of the pixel values in each region.
6. Save the results and image filename to a CSV.

**Technical note:** OpenCV normally reads color images in **BGR** channel order, not RGB. We must convert to RGB before interpreting or displaying values as RGB.

**Deliverable:** A small script that reads a photo and extracts average pixel values from selected regions.

### Phase 3 — Establish the paper/background reference
**When:** Before Monday and then repeat after printing

Measure separate regions of:
- blank ivory paper,
- blank white paper,
- target color on ivory paper, if available.

Record the measured values and compare the paper regions. This helps reveal how different the two paper backgrounds appear in the captured photos.

**Important limitation:** Simply subtracting the paper's RGB values from the printed color's RGB values does not reliably remove the paper's influence. Ink and paper interact, and RGB camera values are not linear physical measurements. We will record paper type and use controlled comparisons rather than assume a simple subtraction fully corrects the difference.

**Deliverable:** A small table of background and target measurements.

### Phase 4 — Generate and print the first A4 chart
**When:** Monday or later, when the JAMK printer is available

1. Generate a Python-created A4 chart with labeled color patches.
2. Start with a small number of patches (for example, 12–20), not a huge chart.
3. Include:
   - a sample ID for every patch,
   - the intended digital RGB or CMYK values,
   - blank-paper regions,
   - repeated copies of a few patches to check consistency.
4. Print on the same type of ordinary A4 white paper, using the same printer and print settings for the experiment.
5. If possible, disable “photo enhancement,” “vivid color,” or automatic color correction and note the settings. Do not assume the printer uses the file's values as exact physical output.
6. Save the source chart and record printer/settings, paper type, and print date.

**Deliverable:** A labeled chart plus a record of the intended digital color values and print settings.

### Phase 5 — Photograph and measure the printed chart

1. Photograph the full chart using the repeatable setup from Phase 1.
2. Load the photo with OpenCV.
3. Correct page perspective if needed; initially, manually selecting patch regions is acceptable.
4. For each patch, sample an inner area and avoid borders, text, shadows, glare, and patch edges.
5. Calculate average color and variability from the sampled pixels.
6. Store one row per sample in a CSV, including:
   - sample ID,
   - intended digital values,
   - measured camera values,
   - paper type,
   - photo filename,
   - date and print settings.

**Deliverable:** A measurement dataset for the printed samples.

### Phase 6 — Compare colors meaningfully

We will use two levels of comparison:

**A. Initial check: RGB**
- Useful for learning and simple diagnostics.
- RGB values depend strongly on the camera, lighting, and processing, so they should not be treated as universal color measurements.

**B. Better comparison: CIELAB and Delta E**
- Convert to CIELAB only with an appropriate, documented color-conversion path.
- Calculate Delta E between the measured sample and the target.
- A smaller Delta E generally means the two measured colors are closer under the chosen calculation.

**Calibration warning:** Converting a phone photo's RGB values directly to Lab does not make the result colorimetrically accurate. Reliable measurement needs controlled lighting and ideally a known color reference chart, a camera/profile calibration workflow, or a colorimeter/spectrophotometer. Until then, report the values as estimates and compare samples captured under the same conditions.

**Deliverable:** A comparison table showing each sample's difference from the target.

### Phase 7 — Test color corrections systematically

The aim is to discover which controlled digital changes move the printed result closer to the target.

1. Choose one target and one baseline sample.
2. Change one factor at a time initially (for example, one channel or one Levels-like adjustment).
3. Print each candidate with an ID and save its intended settings.
4. Photograph and measure all candidates under the same conditions.
5. Calculate each candidate's Delta E from the target.
6. Identify which adjustment improved the result, and by how much.
7. Repeat promising tests to check whether the result is consistent.

Do not assume that “add cyan” or “reduce magenta” can be inferred correctly from a single uncalibrated phone photo. The relationship between digital CMYK settings and printed appearance depends on the printer, ink/toner, paper, driver, and color-management settings.

**Deliverable:** An experiment table mapping adjustments to measured outcomes.

### Phase 8 — Build a data-driven correction recommendation

Only after collecting enough controlled experiments:

1. Define inputs: target measurements, sample measurements, paper type, and print settings.
2. Define the output: an adjustment that has been tested and shown to reduce the color difference.
3. Start with a simple baseline rule or regression model.
4. Split data carefully for validation; avoid evaluating a model only on the samples used to fit it.
5. Compare the model with a simple baseline, such as “make no adjustment.”
6. Report how often the recommendation improves the measured color and by how much.
7. Add complexity only if it demonstrably improves results.

A small collection of arbitrary color patches is not enough to train a reliable general-purpose AI model. The experiments need to be structured and repeated.

**Deliverable:** A validated prototype recommendation, or an evidence-based conclusion that more data/calibration is needed.

### Phase 9 — Evaluate real-world ceramic potential

Before using this for actual ceramic tile production, we would need to validate the method against real tiles and the relevant production process.

Required future evidence may include:
- target tile color measured after firing,
- production sample color measured after firing,
- print settings or CMYK/Levels changes,
- relevant ink, glaze, firing, and surface conditions,
- repeat measurements across batches.

A phone-camera paper prototype can demonstrate image processing and a data-driven experimental workflow, but it cannot establish a reliable correction recipe for fired ceramic tiles on its own.

**Deliverable:** A limitations and validation plan for a future ceramic-focused version.

---

## 5. Proposed project structure

```text
color_measurement_project/
├── data/
│   ├── raw_photos/
│   ├── processed/
│   └── measurements.csv
├── charts/
│   ├── source/
│   └── printed_chart_notes.md
├── notebooks/
│   └── exploration.ipynb
├── src/
│   ├── read_image.py
│   ├── sample_color.py
│   ├── compare_colors.py
│   └── correction_experiments.py
├── reports/
│   └── results.md
└── README.md
```

We do not need to create all these files at the beginning. We will add them gradually as the project grows.

---

## 6. Suggested measurement table

| Column | Meaning |
|---|---|
| `sample_id` | Unique patch or experiment ID |
| `paper_type` | White, ivory, or another paper |
| `target_id` | Reference color being matched |
| `digital_r` / `digital_g` / `digital_b` | Intended digital values, if applicable |
| `digital_c` / `digital_m` / `digital_y` / `digital_k` | Intended CMYK values, if available |
| `measured_r` / `measured_g` / `measured_b` | Average values from the photo |
| `measured_l` / `measured_a` / `measured_b_lab` | Estimated CIELAB values, once implemented |
| `delta_e` | Color difference from the target, once implemented |
| `photo_file` | Original photo filename |
| `lighting_notes` | Light and capture notes |
| `printer_settings` | Printer/driver settings |
| `adjustment_notes` | What was changed in this experiment |
| `repeat_number` | Repeat measurement or print number |

Note: `measured_b` (blue channel) and `measured_b_lab` (CIELAB b* axis) are deliberately named differently to avoid confusion.

---

## 7. Quality checks and common risks

- **Lighting:** Different light can change the apparent color.
- **Auto white balance/exposure:** The phone may make the same paper look different between photos.
- **Paper background:** White and ivory paper are different substrates; a printed color on one will not necessarily match a print on the other.
- **Printer color management:** Digital values are instructions, not guaranteed physical output values.
- **Patch selection:** Avoid borders, text, shadows, and glare.
- **Repeatability:** Repeated photos and prints help distinguish real differences from measurement noise.
- **Target definition:** Preserve the original target; do not redefine the target to match whichever sample was easiest to print.
- **No overclaiming:** Use “camera-estimated color” until the setup is calibrated.
- **Privacy:** Use non-personal images and store only project data.

---

## 8. Work plan for our first sessions

### Session 1 — Before printing
- [ ] Take a photo of blank ivory paper.
- [ ] Take a photo of the target color on ivory paper.
- [ ] Take a photo of blank white A4 paper.
- [ ] Keep original, unedited image files.
- [ ] Record phone and lighting details.
- [ ] Confirm Python environment.
- [ ] Write and understand a small script to load one image and sample a region.

### Session 2 — When the JAMK printer is available
- [ ] Generate the first labeled A4 chart with Python.
- [ ] Print it on ordinary white A4 paper.
- [ ] Record printer and print settings.
- [ ] Photograph the chart consistently.
- [ ] Extract color values and save a CSV.

### Session 3 — Comparison
- [ ] Inspect RGB measurements and variability.
- [ ] Implement the color-space conversion and Delta E comparison.
- [ ] Compare repeated photos and repeated patches.
- [ ] Decide whether measurement quality is sufficient for correction experiments.

### Session 4 — Controlled correction experiments
- [ ] Select a small adjustment strategy.
- [ ] Generate and print candidate samples.
- [ ] Measure candidates under the same conditions.
- [ ] Identify which changes reduced the color difference.
- [ ] Document uncertainty and repeat the promising tests.

### Later sessions
- [ ] Build and validate a correction recommendation.
- [ ] Prepare a clear README and results report.
- [ ] Assess what additional calibration and real ceramic data would be needed.

---

## 9. Definition of success

The first successful milestone is **not** “AI perfectly matches any color.” It is:

1. We can reliably read a photo.
2. We can measure selected color patches consistently enough to compare them.
3. We can compare samples against a preserved target and report the difference.
4. We can run repeatable adjustment experiments.
5. We can demonstrate, with held-out or repeated tests, whether a correction recommendation improves the measured result.
6. We clearly state what is and is not proven for real ceramic production.

This staged approach keeps the project free or low-cost at first, teaches Python and computer vision, and gives us evidence before attempting a more ambitious AI model.
