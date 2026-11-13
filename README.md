# novel-space-object-detection

 
Detecting and identifying novel objects in space by combining **visual data** (images), **orbital data** (TLE histories), and **space-environment data** (space weather, Earth orientation), and then working out what each object is, how it behaves, and what happens to it next.
 
> Items in **[square brackets]** are for the project team to complete. The end-of-life and re-entry module is the only component documented in detail so far. References marked † have a confirmed title and authors, but check year, volume, and pages against the source before citing.
 
## Why combine images and orbital data
 
Public catalogues track a large but incomplete population, and they say little about identity or intent. Images can see objects the catalogue does not list; orbital histories reveal how a catalogued object behaves over time. Neither alone is enough:
 
- **Images** show an object whether or not it is catalogued. In one archival study of wide-field survey images, roughly 20% of verified satellite and debris streaks could not be matched to any catalogued object [1]. Images alone say little about orbit, activity or fate.
- **TLE histories** give the orbit of every catalogued object over years, from which manoeuvres, station keeping, deliberate deorbit and passive decay can be inferred [8, 10-12]. They cannot see what is not in the catalogue.
- **Space-environment data** (solar flux, geomagnetic activity, Earth orientation) is needed to turn orbit changes into physics, since atmospheric drag depends on it [9].
## Pipeline
 
```
 visual data (images)        orbital data (TLEs, catalogue properties)       space environment
        |                                  |                                       |
        v                                  v                                       v
[1] Detection and cross-identification
      detect objects or streaks in images; propagate catalogue TLEs to the image epoch and match;
      unmatched detections are candidate novel objects                                  [describe]
        |
        v
[2] Identification        what is it? spacecraft, rocket body, debris type; from images and orbit   [describe]
        |
        v
[3] Behaviour characterisation   operated, deorbiting or decaying?                    (implemented, see below)
        |
        v
[4] Fate forecasting             future orbit, remaining life, re-entry time and position  (implemented, see below)
```
 
## Data modalities
 
| Modality | Examples | Used for | Status |
|---|---|---|---|
| Visual | [ground-based telescope frames/survey archives / spaceborne camera images: state which] | Detection of objects and streaks, recognition, photometry | [status] |
| Orbital | TLE histories (for example Space-Track), catalogue cross-sections and mass (ESA DISCOS) | Cross-identification, manoeuvre and regime detection, orbit and lifetime forecasting | Implemented for the end-of-life module |
| Space environment | CelesTrak space weather (F10.7, ap), Earth orientation parameters, NRLMSIS 2.0 density | Drag physics and decay forecasting | Implemented for the end-of-life module |
 
## Components
 
| Component | Purpose | Status | Where |
|---|---|---|---|
| Visual detection | [what is detected in images, with which method] | [status] | [`modules/visual-detection/`] |
| Cross-identification | Match image detections to catalogued objects through TLE propagation; flag the unmatched | [status] | [`modules/cross-identification/`] |
| Identification | [what is identified and how] | [status] | [`modules/identification/`] |
| **End-of-life regime detection and re-entry forecasting** | Classify a LEO object as operational, active end-of-life, natural decay or unknown; forecast its orbit, remaining life, and re-entry time and position | LightGBM version run on real objects; transformer version smoke-tested only | `modules/eol-regime-reentry/` |
| Demos and dashboards | Live dashboard, playable simulation page, static reports | Implemented for the end-of-life module | `modules/eol-regime-reentry/` |
 
## Featured module: end-of-life regime detection and re-entry forecasting
 
A physics-informed, multi-task model (LightGBM and a small transformer share one pipeline) that reads an object's TLE history and space-weather record and, causally at each new TLE, outputs:
 
- a regime class: operational (manoeuvring), operational (normal), active end-of-life, natural decay, or unknown;
- predicted perigee and apogee at 13 horizons from 0.25 to 90 days, and the probability of re-entry by each;
- remaining lifetime with a conformal 80% band;
- for objects heading for re-entry, a re-entry time, window and, only when the window is narrow enough to mean something, a position.
Its key signal is a fitted ballistic coefficient from NRLMSIS drag physics [9], which separates station keeping, propulsive descent and passive decay. The models are gradient-boosted trees [14] and a transformer encoder [15], with open-set abstention [16, 17], temperature scaling [19] and conformal calibration [18]. See the module README for the method, installation, commands and results.
 
**Evidence so far.** The LightGBM system was evaluated on four real objects, where the best re-entry-time methods reached a median relative error of about 0.11 to 0.14. The transformer has only been run on a small simulated smoke set. Real-object statistics are indicative because only four objects and a small real active end-of-life class are available.
 
## Repository layout
 
```
modules/
  eol-regime-reentry/    end-of-life regime detection and re-entry forecasting (src/, own README)
  [visual-detection/]    [image-based detection and recognition]
  [cross-identification/][matching detections to the catalogue]
  [identification/]      [object identification]
docs/                    reports, figures and proposals
data/                    raw inputs you provide, not committed
```
 
Each module runs from its own folder, using its own `data/` and `outputs/` directories.
 
## Data
 
Raw data is not redistributed here; check each source's terms before sharing any of it. Sources: TLE histories (for example Space-Track), CelesTrak space weather and Earth orientation parameters, ESA DISCOS object properties, and [the image archives or sensors used].
 
## Getting started
 
1. Clone the repository and pick a module.
2. Follow that module's README. For the end-of-life module:
```bash
cd modules/eol-regime-reentry
pip install -r requirements.txt
python src/predict_mt.py data/tles_44735.txt
```
 
## Roadmap
 
- [ ] [Visual detection and cross-identification milestones]
- [ ] [Identification milestones]
- [ ] Link the modules: feed unmatched image detections into the identification and behaviour stages
- [ ] Train and compare the transformer and LightGBM models on the full simulated set and on real objects
- [ ] Add more real objects and independent labels for the end-of-life module
- [ ] Probabilistic space-weather forecasts for the re-entry interval
## Related work and references
 
**Visual detection, recognition and cross-identification**
 
1. Rachith, E., Hellmich, S., Kneib, J.-P. (2025). Leveraging Astronomical Surveys for Space Debris Detection and Characterization: Application to VST/OmegaCAM. *Advanced Maui Optical and Space Surveillance Technologies Conference (AMOS)*. Streak detection with a CNN and Hough-transform prior, TLE cross-identification, orbit fitting and lightcurves.
2. Stoppa, F., Groot, P. J., Stuik, R., Vreeswijk, P., Bloemen, S., Pieterse, D. L. A., Woudt, P. A. (2024). Automated detection of satellite trails in ground-based observations using U-Net and Hough transform. *Astronomy & Astrophysics*, 692, A199.
3. Nir, G., Zackay, B., Ofek, E. O. (2018). Optimal and efficient streak detection in astronomical images. *The Astronomical Journal*, 156(5), 229.
4. Elhakiem, A. A., Ghoniemy, T. S., Salama, G. I. (2023). Streak detection in astronomical images based on convolutional neural network. *Journal of Physics: Conference Series*, 2616, 012024.
5. Lang, D., Hogg, D. W., Mierle, K., Blanton, M., Roweis, S. (2010). Astrometry.net: blind astrometric calibration of arbitrary astronomical images. *The Astronomical Journal*, 139(5), 1782-1800.
6. Musallam, M. A., Al Ismaeil, K., Oyedotun, O., Perez, M. D., Poucet, M., Aouada, D. (2021). SPARK: SPAcecraft Recognition leveraging Knowledge of Space Environment. arXiv:2104.05978. A synthetic RGB and depth dataset for spacecraft and debris recognition.
7. † *StreakMind: AI detection and analysis of satellite streaks in astronomical images with automated database integration*. arXiv:2605.03429 (2026). [Add authors.]
**Orbital data, drag physics and manoeuvre detection**
 
8. Vallado, D. A., Crawford, P., Hujsak, R., Kelso, T. S. (2006). Revisiting Spacetrack Report #3. AIAA 2006-6753.
9. Emmert, J. T., et al. (2021). NRLMSIS 2.0: A whole-atmosphere empirical model of temperature and neutral species densities. *Earth and Space Science*, 8, e2020EA001321.
10. † Patera, R. P. (2008). Space event detection method. *Journal of Spacecraft and Rockets*, 45(3), 554-559.
11. † Kelecy, T., Hall, D., Hamada, K., Stocker, M. D. (2007). Satellite maneuver detection using two-line element (TLE) data. *AMOS Conference*.
12. † Lemmens, S., Krag, H. TLE consistency check and TLE time-series analysis methods for manoeuvre detection. [Add year and venue.]
13. † Peng, H., Bai, X. (2018). Improving orbit prediction accuracy through supervised machine learning. *Advances in Space Research*, 61(10), 2628-2646.
**Machine-learning methods used**
 
14. Ke, G., et al. (2017). LightGBM: A highly efficient gradient boosting decision tree. *Advances in Neural Information Processing Systems 30*.
15. Vaswani, A., et al. (2017). Attention is all you need. *Advances in Neural Information Processing Systems 30*.
16. Liu, F. T., Ting, K. M., Zhou, Z.-H. (2008). Isolation forest. *IEEE International Conference on Data Mining*, 413-422.
17. Scheirer, W. J., de Rezende Rocha, A., Sapkota, A., Boult, T. E. (2013). Toward open set recognition. *IEEE Transactions on Pattern Analysis and Machine Intelligence*, 35(7), 1757-1772.
18. Romano, Y., Patterson, E., Candes, E. (2019). Conformalized quantile regression. *Advances in Neural Information Processing Systems 32*.
19. Guo, C., Pleiss, G., Sun, Y., Weinberger, K. Q. (2017). On calibration of modern neural networks. *International Conference on Machine Learning*.
## Contributing
 
[Describe how to report issues, propose changes and run tests.]
 
## Citation
 
[Add a citation or `CITATION.cff` once there is a paper or report to cite.]
 
## License
 
[Choose a licence, for example MIT or Apache-2.0, and add a `LICENSE` file.]
 
## Acknowledgements
 
NRLMSIS 2.0, SGP4, CelesTrak and ESA DISCOS. [Add collaborators, image-data providers and funders.]
 
