class: center middle

# BIDS Apps Metadata
### Christopher J. Markiewicz
#### Center for Reproducible Neuroscience
#### Stanford University

###### [effigies.github.io/bids-metadata](https://effigies.github.io/bids-metadata)

---
name: footer
layout: true

<div class="slide-slug">MONII - Montreal, QC - Mar 2019</div>
---
layout: true
template: footer
name: BIDS

# BIDS: Brain Imaging Data Structure

---
layout: true
template: BIDS

.pull-left[![Bids Layout](assets/bids.png)]

---

.pull-right[
* BIDS is a directory structure, not a binary format

* Builds on existing standards (NIfTI, JSON, TSV)

* Intended for human *and* machine legibility

* The [BIDS Validator](https://bids-standard.github.io/bids-validator)
  makes compliance easy to verify

* The [specification](https://bids-specification.readthedocs.io/en/stable/)
  is a searchable HTML document
]

---

.pull-right[

* Basic metadata in the file names

  * Subject, session, imaging modality, etc.
  * Generally just enough to assign unique names

]

---
count: false

.pull-right[
* Basic metadata in the file names
  * Subject, session, imaging modality, etc.
  * Generally just enough to assign unique names

* [`dataset_description.json`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#dataset_descriptionjson),
  [`participants.tsv`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#participants-file),
  [`sessions.tsv`](https://bids-specification.readthedocs.io/en/stable/05-longitudinal-and-multi-site-studies.html#sessions-file),
  and [`scans.tsv`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#scans-file)
  record study-level metadata that may not be associated
  with specific images
]

---
count: false

.pull-right[
* Basic metadata in the file names
  * Subject, session, imaging modality, etc.
  * Generally just enough to assign unique names

* [`dataset_description.json`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#dataset_descriptionjson),
  [`participants.tsv`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#participants-file),
  [`sessions.tsv`](https://bids-specification.readthedocs.io/en/stable/05-longitudinal-and-multi-site-studies.html#sessions-file),
  and [`scans.tsv`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#scans-file)
  record study-level metadata that may not be associated
  with specific images

* NIfTI headers and JSON sidecars contain detailed,
  image-related metadata
]

---
count: false

.pull-right[
* Basic metadata in the file names
  * Subject, session, imaging modality, etc.
  * Generally just enough to assign unique names

* [`dataset_description.json`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#dataset_descriptionjson),
  [`participants.tsv`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#participants-file),
  [`sessions.tsv`](https://bids-specification.readthedocs.io/en/stable/05-longitudinal-and-multi-site-studies.html#sessions-file),
  and [`scans.tsv`](https://bids-specification.readthedocs.io/en/stable/03-modality-agnostic-files.html#scans-file)
  record study-level metadata that may not be associated
  with specific images

* NIfTI headers and JSON sidecars contain detailed,
  image-related metadata

* [Converters](https://bids.neuroimaging.io/#software) make
  populating metadata as painless as possible
]

---
layout: true
template: footer

---

# BIDS Apps and Derivatives

BIDS facilitates *BIDS Apps*, tools that can parse a BIDS directory
structure and operate on the data found.

```Bash
bids-app /bids-directory /output-directory participant [OPTIONS]
```

.footnote[
\* Note that `participant` is an analysis level. Apps may also operate
at the `run`, `session` or `dataset` levels.]

The output of a BIDS App is a *derivative* dataset. The BIDS standard is
[being extended](https://118-151034407-gh.circle-artifacts.com/0/home/circleci/project/site/05-derivatives/01-introduction.html)
to describe many types of derivatives, with a focus on derivatives that
can be reused in yet more BIDS apps.
([Please review and comment](https://github.com/bids-standard/bids-specification/pull/109).)

--

### Hacking opportunities:

* Re-design BIDS App interface
--

* BIDS App startup kit (potential infrastructure in [PyBIDS](https://github.com/bids-standard/pybids/))
--

* BIDS execution specification (more later)

---

# BIDS Derivatives

#### Dataset-level metadata is stored in augmented [`dataset_description.json`](https://118-151034407-gh.circle-artifacts.com/0/home/circleci/project/site/05-derivatives/01-introduction.html#derived-dataset-and-pipeline-description):

* `PipelineDescription` contains references to the code (including version)
  that produced the derivative dataset.
* `SourceDatasets` is a list of references to the specific version of the
  dataset analyzed

--

#### Basic metadata remains in filenames

```
fmriprep/sub-01/func/sub-01_task-rest_space-fsaverage_hemi-L.func.gii
```

--

#### Additional metadata in [sidecar JSON files](https://118-151034407-gh.circle-artifacts.com/0/home/circleci/project/site/05-derivatives/01-introduction.html#common-file-level-metadata-fields)

* `Sources` direct inputs to the process that produced the file
* `CoordinateSystem` indicates the coordinate system of image files
* Metadata of source files that still apply should be preserved
* Metadata of source files that no longer apply must be removed

---
template: footer
layout: true

# BIDS Execution Specification

#### How to record execution details and re-execute?

Consider the command

```Shell
fmriprep /data /out participant \
    --participant-label 01 02 \
    --output-space fsaverage5
```

---

--

Rewrite as a JSON specification:

```JSON
{
  "application": "fmriprep",
  "arguments": ["/data", "/out", "participant"],
  "options": {
    "participant-label": ["01", "02"],
    "output-space": ["fsaverage5"],
  }
}
```

--

If we add in defaults...

---

The specification becomes:

```JSON
{
  "application": "fmriprep",
  "arguments": ["/data", "/out", "participant"],
  "options": {
    "participant-label": ["01", "02"],
    "output-space": ["fsaverage5"],
    "bold2t1w-dof": 6,
    "template": "MNI152NLin2009cAsym",
    "skull-strip-template": "OASIS",
    ...
  }
}
```

---
template: footer

# BIDS Execution Specification

BIDS Apps could permit two modes of execution:

```Bash
bids-app /bids-directory /output-directory participant
```

*or*

```Bash
bids-app --input-spec execution.json
```

Producing as part of the derivatives a *fully* specified `execution.json`:

```
bids-app/
    dataset_description.json
    execution.json
    ...
```

Which can then be used as an input to ensure identical execution in future
uses.

---
layout: true
template: footer

class: center middle

---

### [effigies.github.io/bids-metadata](https://effigies.github.io/bids-metadata)
