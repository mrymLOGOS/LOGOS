# LOGOS
# LOGOS

**An early-stage guardian layer that tries to notice when an autonomous system faces something it has never seen, and hand the decision to a human.**

Status: research prototype. Offline experiments on public data only. Not a certified or deployed safety product.

## The problem

A perception model can be very accurate on what it was trained on and still be confidently wrong on something new. In safety terms these are "unknown unsafe" situations, a category named in ISO 21448 (SOTIF). A system that cannot tell when it is out of its depth cannot hand over to a human in time.

## The idea

LOGOS sits next to a driving system and keeps asking: "Do I recognise this situation?"

1. **Sense:** several independent models look at the same input.
2. **Doubt:** if they disagree, or none is sure, the input is flagged as unfamiliar.
3. **Explain:** a plain-language report says what was flagged and why.
4. **Hand over:** a person confirms the next step while LOGOS recommends a cautious action.

Current status of the four parts: Sense and Doubt are built and tested. Explain is a simple text report. Hand over is a design only.

## Experiments and results

All numbers below come from the notebooks in this repository.

### Prototype 1: handwritten digits (sanity check)

Train on digits 0 to 6, treat 7 to 9 as unknown.

| | Simple model | Guardian (7 models) |
|---|---|---|
| Accuracy on known digits | 99.2% | - |
| Unknowns given a confident (>90%) familiar label | 23.1% | - |
| AUROC | 0.962 | 0.970 |
| Unknowns caught (alarm passes 95% of known) | 74.9% | 81.2% |

Reading: the difference is small on this easy problem. The notebook contains a cell that repeats the comparison over 5 splits.

### Prototype 2: road signs (GTSRB, 32x32 images)

30 sign classes known, 13 unknown (never seen in training).

| | Simple model | Guardian (5 models) |
|---|---|---|
| Accuracy on known signs | 97.2% | - |
| Unknowns given a confident (>90%) familiar label | 51.8% | - |
| AUROC | 0.822 | 0.937 |
| Unknowns caught (alarm passes 95% of known) | 33.4% | 71.9% |

**Fairness check.** Single models from the five scored AUROC 0.789 to 0.848 (25.6% to 40.0% caught). Five models scored by maximum probability reached AUROC 0.938 (73.2% caught), essentially the same as the entropy score (0.937, 71.9%). So the gain comes from using several independent models, a known technique often called deep ensembles. It does not come from the scoring formula.

**Repeated three times** with different random sets of 13 unknown classes (AUROC):

| Run | Simple model | Ensemble (entropy) |
|---|---|---|
| 1 | 0.861 | 0.914 |
| 2 | 0.773 | 0.823 |
| 3 | 0.792 | 0.881 |
| Mean | 0.809 | 0.873 |

The ensemble was better in all three runs, by about 0.05 to 0.09 AUROC.

## Limitations

- Small 32x32 images and offline testing only. No real driving footage or live vehicle.
- Several unknown classes look different from the known ones, which makes detection easier than in real traffic.
- Five models means roughly five times the compute.
- The detection technique itself is established. The project focus is the system around it: explanations for humans, escalation, and sharing alerts across a fleet.
- No safety assessment or certification of any kind.

## Roadmap

1. Real driving data: check whether the gain holds on real road imagery.
2. Harder unknowns: unfamiliar objects that resemble familiar ones.
3. Fleet layer: one vehicle's alert reaches the others in a simulation.
4. Plain-language layer: reports written with a language model.

## Repository layout

```
notebooks/
  LOGOS_prototype_1_digits.ipynb        digits experiment
  LOGOS_prototype_2_road_signs.ipynb    road-sign experiment (needs a GPU)
README.md
```

## How to run

1. Open a notebook in Google Colab (File > Upload notebook, or open from GitHub).
2. For prototype 2, set Runtime > Change runtime type > T4 GPU.
3. Run all cells in order. Prototype 1 takes about a minute. Prototype 2 takes roughly 25 to 40 minutes in total.

Dependencies: numpy, matplotlib, scikit-learn, PyTorch, torchvision (all preinstalled in Colab).

## Background and references

- ISO 21448 (SOTIF) overview: https://ansys.synopsys.com/simulation-topics/what-is-sotif
- UL 4600 overview: https://www.jamasoftware.com/blog/understanding-ul-4600-ensuring-safety-for-autonomous-products/
- Dataset: German Traffic Sign Recognition Benchmark (GTSRB), loaded through torchvision.

LOGOS has not been assessed against either standard. They are cited only to describe the problem area.

## Contact

[MRYMLOGOS] آ·

[email] آ· mrymlogos@gmail.com

[LinkedIn]    www.linkedin.com/in/mrymLOGOS


