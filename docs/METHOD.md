# Method

## 1. Notation

Each rectified text line is a strip of height *H* = 84 px with baseline at *y_b* = 62 px. The x-height is normalized to 15 px. A glyph is a tuple (α, e, x), where:

- α ∈ [0,1]^{H×w} is an ink-opacity map;
- e and x are the entry and exit anchors, defined as heights at the left and right borders where the stroke meets the median zone.

## 2. Ink segmentation

- Ink opacity is estimated as α = clip((BH(I) − τ)/κ, 0, 1), where BH is the morphological black-top-hat of the luminance channel. For ballpoint specimens, the red channel is used, since it maximizes contrast for blue ink.
- Printed rulings are removed with horizontal morphological opening.
- Strokes darker than the ruling are restored where the two overlap.

## 3. Line rectification

Ruling pixels are clustered per line, and a low-order polynomial y = p(x) is fitted to each cluster. Each line is then resampled with `remap` onto a straight baseline. The inverse map is used at render time, so synthesized text follows the real curvature of the paper.

## 4. Template-guided segmentation

For a word with characters c₁…c_n and candidate cut positions P, we maximize

  Σᵢ ⟨φ(α[p_{i-1}:p_i]), T_{cᵢ}⟩ − λ ((w_i − k·ŵ_{cᵢ}) / k·ŵ_{cᵢ})² − μ·ink(p_i)

by dynamic programming, where:

- φ is a zoned, band-averaged column descriptor;
- T_c is the class template;
- ŵ_c is the expected glyph width;
- k is the per-word scale.

Templates are re-estimated from the best 70% of assignments, and the process is iterated (EM-style). Three iterations suffice in practice.

## 5. Synthesis

- **Whole words.** Whole-word specimens are preferred.
- **Out-of-vocabulary words.** Glyphs are drawn uniformly from the top-k per class (k = 6, chosen by ablation).
- **Joining.** Consecutive glyphs are joined by a quadratic Bézier connector from the exit anchor of one glyph to the entry anchor of the next. The connector starts at the baseline, or at the top for the classes {o, v, w, b, r}.
- **Descenders.** Glyphs with descenders {g, j, p, q, y, f} are anchored below the baseline.
- **Per-word jitter.** Each word receives rotation in [−0.9°, 0.9°], a baseline offset in {−1, 0, 1} px and a pressure factor in [0.86, 1].

## 6. Pen rendering

The synthesized opacity is skeletonized (Zhang–Suen) and re-dilated with a constant 3 px nib. It is then modulated by low-frequency pressure noise and composited over a photographed blank sheet registered by projective rectification of its printed frame.

## 7. Trajectory planning

- **Tracing.** Skeleton pixels are traced into 8-connected polylines, starting from endpoints, and ordered left to right.
- **G-code emission.** Each polyline is emitted as `G0` (travel, pen up), servo pen-down, `G1` segments, then pen-up.
- **Units and coordinates.** Units are millimetres, with absolute coordinates.
- **Simulation first.** Simulation precedes any physical motion.

## 8. Evaluation

Legibility is assessed by HOG-based word spotting:

1. For each test word, the word specimen and all glyphs cut from it are excluded.
2. The word is synthesized from the remaining glyphs.
3. The synthesized word is ranked against all real word specimens.

We report top-1 and top-5 retrieval rates. The reference ceiling is the real-versus-real self-retrieval rate.
