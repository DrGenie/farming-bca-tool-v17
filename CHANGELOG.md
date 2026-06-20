SOIL CRC Benefit-Cost Analysis Tool - Changelog
================================================

Version 2026.16 (v16)
---------------------
Summary: Improved Analysis Assistant response visibility, removed the duplicate
View results control, clarified navigation and stale-result states, improved
mobile usability, corrected visible version information and strengthened
accessibility. Scientific BCA calculations were preserved and regression-tested.

Analysis Assistant (response area always visible)
- Rebuilt the panel order: header + three status badges, one introductory
  sentence ("Ask me to explain, compare or draft text using the current
  analysis."), the large conversation/response area, four primary quick prompts,
  a collapsed "More prompts", then a sticky input and the small Clear/Download
  controls. Longer explanations, privacy text and examples moved into a collapsed
  "About this assistant".
- The response area now uses flex:1 / min-height:0 / overflow-y:auto so it is the
  largest region, scrolls independently, never collapses and is never covered by
  the input. Generous bottom padding; auto-scrolls to the newest message;
  "Thinking…" shows inside the conversation. The input bar is sticky with an
  opaque background and a top border. On mobile the panel is full-screen using
  100dvh and the composer clears the on-screen keyboard / home indicator; the
  floating launcher hides while the panel is open.
- Three concise, non-colour-only status badges: Backend (Connected/Checking/Not
  set/Issue), Analysis (Current/Not run/Out of date) and Data/Treatments
  ("N ranked" / "No treatments ranked" / "Action required: Run analysis again").
- "More prompts" is collapsed by default and auto-collapses after a secondary
  prompt. All quick prompts (primary and secondary) and Send are disabled during
  generation; Stop is shown; controls restore on success, error or cancellation.
- Distinct re-run messages: new data loaded vs settings changed vs nothing
  analysed. The assistant never quotes stale numbers; tool state now also carries
  staleReason, settingsVersion and a settings signature.

Navigation (one logical path)
- Removed the duplicate, inactive "View results" button from the upload action
  row. The top row now holds only Download template, Upload workbook and Load
  sample data.
- The single lower control is renamed "Continue to results" and becomes active
  only when dataLoaded && dataValidated (essential validation passed). It opens
  the Results tab; it does not imply results already exist. After a successful
  load it says "The workbook has been loaded and validated. Continue to Results
  to run the Benefit-Cost Analysis."; if validation blocks, it stays disabled with
  "Resolve the validation errors above before continuing."

Results state messaging and confirmation
- The Results banner now states the four states explicitly (no data / loaded not
  run / stale / current) and offers an inline "Run Benefit-Cost Analysis now".
- Upload confirmation shows the active data source, filename or sample label,
  worksheet, counts, control, load date/time, validation status, and explicitly
  that previous results were cleared and the BCA must be re-run. Announced via an
  ARIA live region.

Results tables
- Ranking table: horizontal scroll, sticky Rank + Treatment columns, scoped
  headers with units, a Compact view (Treatment, NPV, BCR, Difference versus
  control) and a Detailed view (all metrics), and a "Download ranking (CSV)".
- A note clarifies that total NPV is not the same as incremental NPV versus the
  control.

Formula audit, sensitivity, report
- "How the calculations work" is collapsed by default with a brief visible
  summary; the full formulas/units/components and active run details remain
  inside.
- Sensitivity shows the selected treatment prominently, uses named scenarios
  (Conservative / Alternative / Optimistic), adds Reset to defaults, and refuses
  to update on stale results.
- The report's external ChatGPT/Copilot drafting moved into a collapsed
  "External AI drafting (optional)", kept separate from the integrated assistant;
  AI narrative is labelled draft text for review.

Version and cache
- All current-release labels now read Version 2026.16 / v16, and the contact line
  asks users to quote Version 2026.16. Asset links are cache-busted
  (styles.css?v=16-1, app.js?v=16-1, assistant.js?v=16-1).

Scientific integrity (unchanged and re-verified)
- No calculation, discounting, cost-summation, aggregation, ranking, comparison
  or sensitivity formula was changed. verify.js reproduces the v13/v15 figures
  exactly (Manure T11 NPV $6,639.01/ha, BCR 2.056; control T00 $4,094.41/ha,
  BCR 1.627; 12 treatments) and confirms full dataset replacement. All checks
  pass.


Version 2026.13.1 (v13.1)
-------------------------
Summary: Targeted corrective release. Fixed the workbook-replacement and
state-propagation problems so that newly loaded data (sample or uploaded)
completely replace any previous dataset across validation, calculations,
treatment selectors, charts, sensitivity, reports and the Analysis Assistant.
Added explicit stale-result controls. The scientific BCA formulas are unchanged
and were regression-tested against v13 (identical results).

Scientific integrity (unchanged and re-verified)
- No calculation, discounting rule or cost-summation logic was altered. The
  automated parity harness (verify.js) reproduces the v13 headline figures
  exactly from the official sample workbook and default settings: top treatment
  Manure (T11) NPV $6,639.01/ha, BCR 2.056; control Control (T00) NPV
  $4,094.41/ha, BCR 1.627; 12 treatments ranked.

Fixed: dataset replacement and stale state (root cause)
- Introduced a single source of truth on the existing application-state object
  with explicit lifecycle fields: dataSource (none/sample/upload), datasetVersion,
  analysisDatasetVersion, dataLoaded, dataValidated, settingsApplied, analysisRun
  and resultsStale, plus a resultsAreCurrent() guard.
- Added a mandatory resetAnalysisForNewDataset() that runs BEFORE any load. It
  clears all derived data (cleaned rows, grouped treatments, ranking, sensitivity,
  validation, selected treatments/control/side-by-side, last run, report HTML and
  narrative), resets every output (metric cards, ranking table, comparison,
  interpretation, calculation-audit schedule, sensitivity table), empties the
  treatment selectors, and clears the charts (inline SVG containers, so the
  previous chart is destroyed when its markup is replaced).
- Every load now increments datasetVersion. handleWorkbook resets first, bumps the
  version, parses, rebuilds selectors from the uploaded workbook only, and no longer
  auto-runs the analysis: results stay unavailable until the user clicks Run, so
  sample and uploaded data can never be mixed in a displayed result.
- runAnalysis stamps analysisDatasetVersion = datasetVersion and clears resultsStale.
  Results, report and exports are blocked (with a clear "re-run required" message)
  whenever analysisDatasetVersion does not match datasetVersion.
- Changing a setting after a run marks results stale; applying settings re-runs and
  re-stamps the version, so old results are never shown under new settings.

Fixed: failed-upload policy (no ambiguity)
- A workbook that cannot be parsed CLEARS the dataset (the reset has already removed
  the previous data and outputs). The tool states that no dataset is active and asks
  for a valid upload; invalid data are never shown as successfully loaded, and the
  tool never silently falls back to the sample data.

Added: verification that uploaded data replace sample data
- Visible "Active data source: ..." label on the Data tab and at the top of the
  Results tab, plus a Results stale/empty banner.
- window.getBcaToolState() now reports dataSource, datasetVersion,
  analysisDatasetVersion, dataLoaded, dataValidated, analysisRun, resultsStale and
  analysisValid, and WITHHOLDS the ranking and sensitivity payload whenever results
  are not current, so no consumer can cite stale numbers.

Improved: Analysis Assistant integrity
- The assistant trusts analysisValid/hasResults. If new data are loaded or settings
  changed since the last run, it answers "New data have been loaded ... please run
  the BCA again" and does not quote earlier numbers. If nothing has been analysed it
  answers "Please upload data and run the BCA first". The larger, independently
  scrollable response area (min-height enforced, near full-screen on mobile, fixed
  input) is retained and reinforced so responses stay visible during and after
  generation.

Automated tests (verify.js, run with node)
- Calculation parity: PASS (v13 figures reproduced exactly).
- Dataset replacement: load sample, then workbook 2 (3 treatments, different names,
  different control, different yields/costs) — PASS: no sample treatment or value
  survives; all outputs change.
- State-contract checks against the shipped app.js/assistant.js source — PASS.


Version 2026.13 (v13)
---------------------
Summary: Improved accessibility, result interpretation, assistant layout, report
clarity, validation messages and mobile usability. Scientific BCA calculations
preserved and verified against v12.

Calculation integrity
- No change to any calculation. PV of benefits, PV of costs, NPV, BCR, gross
  profit margin, ROI, difference versus control, treatment ranking, control and
  side-by-side comparison, sensitivity, discounting modes and cost-component
  summation are byte-for-byte identical to v12.
- Verified automatically: the same sample workbook and default settings produce
  the same results as v12 (for example, the top treatment "Manure" at an NPV of
  $6,639.01/ha and a BCR of 2.056, with the control "Control" (T00) at an NPV of
  $4,094.41/ha). A before/after snapshot of every metric was diffed and is
  identical.

New: formula transparency and calculation audit
- Added a "How the calculations work (calculation audit)" panel in Results,
  listing plain-language formulas and units:
  - Annual gross benefit = average yield (t/ha) x grain price ($/t)
  - Annual direct cost = sum of the individual cost components ($/ha)
  - PV benefits / PV costs = discounted values over the analysis period
  - NPV = PV benefits - PV costs
  - BCR = PV benefits / PV costs
  - Gross profit margin (%) = (PV benefits - PV costs) / PV benefits x 100
  - ROI (%) = (PV benefits - PV costs) / PV costs x 100
  - Difference versus control = selected treatment NPV - control NPV
- States explicitly that direct costs are summed from the individual workbook
  components (not a single pre-totalled column) and lists those components.
- Shows the active discount schedule and assumptions after each run.

New: result interpretation
- Added a "What these results suggest" panel that turns the live numbers into
  plain language for NPV, BCR, ROI, the comparison with the control and why the
  top treatment ranks where it does. Uses careful language ("suggests", "under
  these assumptions", "based on the uploaded data") and a check-assumptions
  caveat. Not presented as financial or agronomic advice.

Improved: data validation and warnings
- Validation messages now state what is wrong, where the problem likely is, how
  to fix it, and whether the analysis can continue. Covers wrong sheet, missing
  BCA Data sheet, missing/duplicated control, inconsistent treatment names,
  missing or non-numeric yield, missing direct-cost components and non-numeric
  cost cells.
- Added a "Ready to analyse" indicator that turns green once essential checks
  pass, amber when results can be produced but need review, and red when a
  control or yield is missing.

Improved: layout and user flow
- Added a step-by-step progress strip across the top: Upload data, Validate
  workbook, Project details, Apply settings, Run analysis, Compare treatments,
  Run sensitivity, Generate report. The current step is highlighted and
  completed steps are ticked.
- Units shown beside inputs and outputs ($/t, years, %, $/ha, t/ha).

Improved: accessibility
- Skip link, ARIA live regions, visible and programmatic labels retained.
- Charts are now accompanied by accessible data tables and plain-language
  summaries so they can be read in greyscale or by a screen reader (colour is
  never the only cue).
- Ranking table headers use scope="col". Progress and ready states are announced
  via aria-live / aria-current.

Improved: Analysis Assistant
- Simplified to four primary quick buttons (Explain current result, Explain for
  a farmer, Best value for money, Draft report narrative). The eight secondary
  prompts moved into a collapsed "More prompts" section.
- Larger, independently scrollable response area; input fixed at the bottom;
  near full-screen sheet on mobile.
- If the analysis has not been run, result questions now answer locally with:
  "Please upload data and run the BCA first so I can interpret the current
  results." General questions (what the metrics mean, etc.) still work.
- Specific backend error messages retained: backend unavailable, API key missing
  or invalid, quota or rate limit reached, model unavailable, CORS problem,
  question too long, timeout and empty response. API keys never appear in the
  browser; the frontend only calls the Worker.

Improved: report generation
- Report section headings (for example "Description of Project:") render as bold
  labels with a colon, with the user-written text in a separate, non-bold
  paragraph.
- Locally generated or AI-generated narrative is labelled as draft text to be
  reviewed, with a disclaimer that outputs do not replace professional advice.

Trust and transparency
- Added a "What this tool can and cannot do" section to the Introduction.

Workbook and template changes
- The downloadable template is BCA_template.xlsx and includes the
  "Cost of amendment_per_ha" column (Cost of Amendments).
- The sample/trial workbook is BCA_Trial_data.xlsx. The cost of amendment is set
  to zero, previously negative figures have been corrected, and the
  "Total Farm Costs_per_ha" column is recalculated as a live sum of the same
  direct-cost components the tool uses, so the workbook total matches the tool's
  internal direct-cost total.
- The tool reads cost components directly from the workbook; the total farm cost
  column is shown for reference only and does not affect the analysis.

Compatibility
- Still a static site: open index.html or deploy the folder to GitHub Pages. No
  build step. The optional Analysis Assistant uses a Cloudflare Worker that
  holds the model key as a server secret (never in the frontend).
