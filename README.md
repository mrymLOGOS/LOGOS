LOGOS
An early-stage guardian layer that tries to notice when an autonomous system faces something it has never seen, or is drifting toward failure, explain why in plain language, and leave the decision to a human.

Status: research prototype. Offline experiments only (public datasets and simulation). Not a certified or deployed safety product.

The problem
A perception model can be very accurate on what it was trained on and still be confidently wrong on something new. In safety terms these are "unknown unsafe" situations, a category named in ISO 21448 (SOTIF). A system that cannot tell when it is out of its depth, or when its health is drifting toward failure, cannot hand over to a human in time.

The idea
LOGOS sits next to a driving system and keeps asking two questions: "Do I recognise this situation?" and "Is this heading toward failure?"

Sense: several independent models look at the same input.
Doubt: if they disagree, or none is sure, the input is flagged as unfamiliar.
Anticipate: signals over the last few seconds are used to estimate whether failure is approaching.
Explain: a plain-language report says what was flagged and why.
Hand over: a person confirms the next step while LOGOS suggests a cautious action.
Current status: Sense and Doubt are built and tested (prototypes 1 and 2). Anticipate is tested in simulation and on NASA engine data (prototypes 3 and 4). Explain is a text report on the engine data. Hand over is a simulated approve/reject prompt only.

Experiments and results
All numbers below come from the notebooks in this repository.

Prototype 1: handwritten digits (sanity check)
Train on digits 0 to 6, treat 7 to 9 as unknown.

Simple model	Guardian (7 models)
Accuracy on known digits	99.2%	-
Unknowns given a confident (>90%) familiar label	23.1%	-
AUROC	0.962	0.970
Unknowns caught (alarm passes 95% of known)	74.9%	81.2%
Reading: the difference is small on this easy problem.

Prototype 2: road signs (GTSRB, 32x32 images)
30 sign classes known, 13 unknown (never seen in training).

Simple model	Guardian (5 models)
Accuracy on known signs	97.2%	-
Unknowns given a confident (>90%) familiar label	51.8%	-
AUROC	0.822	0.937
Unknowns caught (alarm passes 95% of known)	33.4%	71.9%
Fairness check. Single models from the five scored AUROC 0.789 to 0.848 (25.6% to 40.0% caught). Five models scored by maximum probability reached AUROC 0.938 (73.2% caught), essentially the same as the entropy score (0.937, 71.9%). So the gain comes from using several independent models, a known technique often called deep ensembles. It does not come from the scoring formula.

Repeated three times with different random sets of 13 unknown classes (AUROC): simple model mean 0.809, ensemble mean 0.873. The ensemble was better in all three runs, by about 0.05 to 0.09.

Prototype 3: failure-path prediction (simulation)
Simulated drives: a hidden stress level rises slowly before a failure, harmless short spikes (glare, tunnels) occur, and 25% of failures are abrupt with no warning. Every method is set to the same 4 false alarms per healthy hour; 5 repeats, mean (+- std). Share of failures warned in the last 60 s:

Harmless spikes	Single-reading threshold	Trend model (logistic)	Trend model (boosting)
No	71% (+-3)	75% (+-3)	73% (+-3)
Yes	22% (+-2)	75% (+-3)	74% (+-4)
Reading: with harmless spikes a single reading fails while a 20 s trend holds up. Without spikes the difference is within noise. The roughly 25% not warned are the abrupt faults, which no method can foresee. The data is simulated by us, so this result partly follows from the simulator's design.

Prototype 4: real run-to-failure data (NASA C-MAPSS) and a readable report
NASA turbofan engine degradation data (a NASA simulation of jet engines, not cars). Engines are split by engine (5-fold, 2 repeats), every method is set to the same false-alarm rate, and an engine counts as warned if an alarm fires in its last 60 cycles.

FD001 (simple conditions): all methods warned 100% of engines; models using many sensors warned about 10 cycles earlier than the best single sensor.
FD002 (six operating conditions): with a strict budget (1 false alarm per 1000 healthy cycles) the best single sensor warned only 39% of engines; all multi-sensor models warned 100%.
A model using only current readings of all sensors did about as well as the trend models. In this data the gain came from combining sensors, not from trends. The trend advantage from prototype 3 was not reproduced on real data.
The notebook also prints an alert report: score, the three sensors that pushed it up, how far they are from early-life values, suggested action, limits, and a human approve/reject prompt (which only records the answer).

Limitations
Offline testing only. No real driving footage, no live vehicle. Prototype 4 data is a NASA simulation of jet engines.
Prototype 2 images are small (32x32), and several unknown classes look different from the known ones, which makes detection easier than in real traffic.
Prototype 3 data is simulated by us.
Predicting failure from sensors (remaining useful life) and ensemble-based unknown detection are established techniques. The project focus is the system around them: explanations for humans, escalation, and knowing the limits of what the system knows.
The warning score is not a calibrated probability. Some failures give no warning at all.
No safety assessment or certification of any kind.
Roadmap
Real driving data: check whether the gains hold on real road imagery and real vehicle logs.
Harder unknowns: unfamiliar objects that resemble familiar ones.
More engine data (FD003, FD004) and confidence intervals on the comparison.
Fleet layer: one vehicle's alert reaches the others in a simulation.
Plain-language layer with a language model, checked against the evidence behind each alert.
Repository layout
notebooks/
  LOGOS_prototype_1_digits.ipynb          digits experiment
  LOGOS_prototype_2_road_signs.ipynb      road-sign experiment (needs a GPU)
  LOGOS_prototype_3_failure_path.ipynb    failure-path simulation (CPU)
  LOGOS_prototype_4_real_engines.ipynb    NASA engine data + readable report (CPU)
README.md
How to run
Open a notebook in Google Colab (File > Upload notebook, or open from GitHub).
For prototype 2, set Runtime > Change runtime type > T4 GPU. Prototypes 1, 3 and 4 run on CPU.
Run all cells in order. Approximate times: prototype 1 about a minute, prototype 2 25 to 40 minutes, prototype 3 a few minutes, prototype 4 a few minutes.
Dependencies: numpy, pandas, matplotlib, scikit-learn, PyTorch, torchvision (all preinstalled in Colab).

Background and related work
ISO 21448 (SOTIF) overview: https://ansys.synopsys.com/simulation-topics/what-is-sotif
UL 4600 overview: https://www.jamasoftware.com/blog/understanding-ul-4600-ensuring-safety-for-autonomous-products/
Datasets: German Traffic Sign Recognition Benchmark (GTSRB, via torchvision); NASA C-MAPSS turbofan engine degradation simulation, https://data.nasa.gov/dataset/simulating-degradation-data-for-prognostic-algorithm-development
Related work on runtime out-of-distribution detection and safety cages for driving (we have not compared against these): Connected Dependability Cage, arXiv 2604.27728; OoDIS benchmark for anomaly instance segmentation, arXiv 2406.11835.
LOGOS has not been assessed against any standard. They are cited only to describe the problem area.

Contact
[Name] · [email] · [LinkedIn]
