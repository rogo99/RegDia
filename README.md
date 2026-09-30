# Regensburger Diarium — Historical Document Transcription and Entity Extraction

This repository contains the code used to transcribe and extract structured information from historical issues of the **Regensburger Diarium** using a vision-language model (VLM).

The pipeline is designed for historical German printed documents, including **18th-century Fraktur print**, and consists of two main stages:

1. **Image transcription:** page images are segmented into vertical sections and transcribed by `Qwen/Qwen2.5-VL-7B-Instruct`.
2. **Entity extraction:** the generated transcriptions are processed in page chunks and passed to the same VLM to extract structured events and entities as JSON.

The repository also contains evaluation code for comparing the generated results with manually annotated reference data.

---

## Pipeline

The complete workflow is:

```text
Historical page images
        │
        ▼
Image preprocessing
        │
        ├── grayscale conversion
        ├── Otsu binarization
        ├── text-region detection
        └── content-aware vertical splitting
        │
        ▼
Page tiles
        │
        ▼
Qwen2.5-VL-7B-Instruct
        │
        ▼
Historical transcription
        │
        ▼
Transcription chunks
        │
        ▼
Qwen2.5-VL-7B-Instruct
        │
        ▼
Structured JSON extraction
        │
        ▼
Issue-level JSON files
        │
        ▼
Evaluation
```

The transcription stage is intentionally instructed to preserve historical spelling and wording rather than modernizing the text.

The extraction stage identifies information such as:

* baptisms
* burials
* deaths
* arrivals and departures
* persons
* occupations
* dates
* ages
* gates
* directions
* transport
* origins
* lodgings
* traveller reference values (`s_value`)

---

## Model

The pipeline uses:

**Qwen/Qwen2.5-VL-7B-Instruct**

The model is loaded with **4-bit NF4 quantization** and FP16 computation:

```python
BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.float16,
    bnb_4bit_quant_type="nf4"
)
```

The experiments were conducted in **Google Colab** using an **NVIDIA Tesla T4 GPU with 15 GB of GPU memory**.

A GPU is strongly recommended for running the pipeline. The 7B model is loaded in 4-bit mode to reduce GPU memory requirements.

---

## Requirements

The notebook was developed for **Google Colab**.

The required Python packages are installed directly in the notebook:

```python
!pip install -U "bitsandbytes>=0.46.1" accelerate transformers
```

and:

```python
!pip install json_repair
```

Therefore, no separate local Python environment or manual installation of these packages is required when running the notebook in Colab.

### Additional requirements

You also need:

* a Google account with access to Google Colab
* access to a (free) GPU runtime in Colab
* Google Drive for the input and output files
* the historical page images to be processed

The Qwen model is downloaded automatically from Hugging Face when the notebook executes:

```python
Qwen/Qwen2.5-VL-7B-Instruct
```

No manual model download is required.

---

## Google Colab GPU

Before running the model, make sure that a GPU runtime is enabled in Google Colab:

**Runtime → Change runtime type → Hardware accelerator → GPU**

The exact GPU provided by Google Colab can vary between sessions. The experiments for this project used an NVIDIA Tesla T4 with 15 GB of GPU memory.

You can check the GPU available in the current session with:

```bash
!nvidia-smi
```

---

## Directory configuration

**The directory paths in the notebook are specific to the original development environment and must be changed before running the code.**

In particular, change paths such as:

```python
base_dir = "/content/drive/MyDrive/Colab Notebooks/RegDia_pages"
```

```python
output_base = "/content/drive/MyDrive/Colab Notebooks/RegDia_transcriptions"
```

and:

```python
transcription_base = "/content/drive/MyDrive/Colab Notebooks/RegDia_transcriptions"
```

```python
entity_base = "/content/drive/MyDrive/Colab Notebooks/RegDia_entities"
```

to the directories used in your own Google Drive.

The paths do not have to use the same folder names as the original project. What matters is that the input and output locations correspond to the expected directory structure described below.

---

## Input directory structure

The page images are expected to be organized into one directory per issue:

```text
RegDia_pages/
├── Num_1_02_01_1770/
│   ├── bsb11130457_00001.jpg
│   ├── bsb11130457_00002.jpg
│   └── ...
├── Num_2_09_01_1770/
│   ├── bsb11130457_00010.jpg
│   ├── bsb11130457_00011.jpg
│   └── ...
└── ...
```

Issue folder names must follow this pattern:

```text
Num_<issue_number>_<day>_<month>_<year>
```

For example:

```text
Num_2_09_01_1770
Num_31_31_07_1770
Num_52_24_12_1770
```

The code extracts the issue number and date directly from the folder name.

---

## Page filenames

Page filenames must contain a numeric canvas identifier before the `.jpg` extension.

For example:

```text
bsb11130457_00010.jpg
bsb11130457_00011.jpg
bsb11130457_00012.jpg
```

The code extracts the canvas number from the final numeric part of the filename.

For example:

```text
bsb11130457_00010.jpg
```

is interpreted as canvas:

```text
10
```

---

## Step 1: Mount Google Drive

The notebook begins by mounting Google Drive:

```python
from google.colab import drive
drive.mount('/content/drive')
```

Authorize Google Colab when prompted.

After mounting Google Drive, make sure that the directory paths in the notebook point to the correct locations.

---

## Step 2: Install dependencies

Run:

```python
!pip install -U "bitsandbytes>=0.46.1" accelerate transformers
```

and:

```python
!pip install json_repair
```

The notebook then imports the required libraries and loads the Qwen model.

---

## Step 3: Load the model

The model is loaded using:

```python
model_name = "Qwen/Qwen2.5-VL-7B-Instruct"
```

with 4-bit NF4 quantization.

The model is automatically downloaded the first time it is loaded. Depending on the Colab environment and network connection, this step can take some time.

---

## Step 4: Transcription

For each page, the code:

1. loads the JPG image;
2. converts it to grayscale for layout analysis;
3. applies Otsu binarization;
4. detects horizontal text regions;
5. identifies whitespace gaps near one-third and two-thirds of the page;
6. uses these gaps to divide the page into up to three vertical tiles;
7. resizes each tile to a width of 1024 pixels;
8. sends each tile to Qwen2.5-VL;
9. combines the tile transcriptions in their original vertical order.

The VLM is instructed to:

* transcribe the entire page;
* preserve historical spelling;
* preserve line breaks;
* preserve the section structure;
* not summarize;
* not modernize spelling;
* mark unclear words as `[unclear]`.

The resulting transcription is saved as a `.txt` file.

For example:

```text
RegDia_transcriptions/
└── Num_2_09_01_1770/
    ├── bsb11130457_00010.txt
    ├── bsb11130457_00011.txt
    └── ...
```

Each transcription also contains metadata for the issue number, issue date, and canvas number.

---

## Step 5: Entity extraction

The transcription files are then processed by the extraction stage.

To reduce GPU memory usage, the code processes a maximum of:

```python
CHUNK_SIZE = 4
```

transcription pages at a time.

For each chunk, the transcription is passed to Qwen2.5-VL together with a detailed extraction prompt defining the required JSON structure.

The model extracts information from the complete transcription rather than only from individual sentences.

The output contains three main sections:

```json
{
    "evangelisch": {
        "parishes": []
    },
    "catholisch": {
        "parishes": []
    },
    "arrivals_departures": []
}
```

The code then adds issue metadata programmatically.

---

## Step 6: JSON handling

The model is instructed to return valid JSON.

If the model nevertheless produces malformed JSON, the code attempts:

1. normal JSON parsing;
2. extraction of a JSON object from surrounding text;
3. JSON repair using `json_repair`.

`json_repair` is only used to repair JSON syntax. It does not perform semantic correction, entity normalization, or deduplication.

The chunk results are then merged in their original order.

A final issue-level JSON file is created only if **all chunks of the issue were processed successfully**.

Example:

```text
RegDia_entities/
└── Num_2_09_01_1770/
    ├── Num_2_09_01_1770_chunk_1.json
    ├── Num_2_09_01_1770_chunk_2.json
    └── Num_2_09_01_1770.json
```

---

## Existing output files

The entity-extraction stage checks whether the final issue JSON already exists.

If:

```text
Num_2_09_01_1770.json
```

already exists, that issue is skipped.

This prevents already completed issues from being processed again.

If an issue needs to be processed again, remove or rename its existing final JSON file before rerunning the extraction stage.

---

## Evaluation

The final part of the notebook contains evaluation code for comparing generated JSON files with manually annotated reference data.

---

## Output

The main outputs are:

### Transcriptions

```text
RegDia_transcriptions/
└── <issue>/
    └── <canvas>.txt
```

These contain the VLM-generated historical transcriptions.

### Entity extraction

```text
RegDia_entities/
└── <issue>/
    ├── <issue>_chunk_1.json
    ├── <issue>_chunk_2.json
    └── <issue>.json
```

The final issue-level JSON contains the structured extraction and source metadata.
Amount of chunks may vary based on the input.

---

## Important notes

### Historical spelling

The transcription and extraction stages are designed to preserve historical wording. The pipeline does not perform general semantic modernization of the extracted information.

### No semantic post-correction

After the VLM generates the extraction, the code does not manually correct or normalize the extracted entities. The `json_repair` package is only used for syntactic JSON repair.

### Memory requirements

The model is a 7B-parameter VLM. The notebook therefore uses 4-bit quantization and processes extraction input in chunks of four pages.

If the available GPU has substantially less memory, the chunk size may need to be reduced.

### Runtime

Processing the complete corpus can take a considerable amount of time because every page is transcribed and the resulting transcriptions are subsequently processed by the VLM for entity extraction.

### Reproducibility

The exact GPU available in Google Colab can vary. Model and package versions may also change over time. For reproducibility, record the runtime configuration and package versions when conducting a new experiment.

---

## Repository structure

A possible repository structure is:

```text
.
├── README.md
├── <notebook>.ipynb
├── data/
│   └── ...
├── RegDia_transcriptions/
│   └── ...
├── RegDia_entities/
│   └── ...
└── evaluation/
    └── ...
```

Large image collections and generated files do not necessarily need to be included directly in the Git repository. Their location can instead be configured through the directory paths in the notebook.

---

## Quick start

1. Open the notebook in **Google Colab**.
2. Enable a **GPU runtime**.
3. Mount Google Drive.
4. Place the Regensburger Diarium page images in the required issue-folder structure.
5. **Change all directory paths in the notebook** to match your own Google Drive.
6. Run the dependency installation cells.
7. Run the model-loading cell.
8. Run the transcription pipeline.
9. Run the entity-extraction pipeline.
10. Inspect the generated `.txt` and `.json` files.
11. Run the evaluation code if reference annotations are available.

The model and output directories can be changed independently as long as the paths used by each subsequent processing stage point to the corresponding outputs of the previous stage.


