# MGR-KD

MGR-KD is an LLM-guided framework for knowledge discovery over a typed sports knowledge graph. The implementation contains four components:

1. **Multi-granularity sports knowledge representation** combines entity, relation, type, and graph-neighborhood signals.
2. **Query-adaptive hierarchical graph retrieval** restricts candidates with the relation schema and ranks them with structural and path evidence.
3. **LLM-guided graph reasoning** plans relation paths and reranks only graph-retrieved candidates.
4. **Confidence-aware discovery and verification** fuses graph, path, textual, schema, and LLM evidence.

The repository intentionally excludes datasets, checkpoints, caches, logs, figures, and experimental results.

## Repository layout

```text
mgr-kd/
├── configs/default.yaml
├── src/mgr_kd/
│   ├── data.py
│   ├── model.py
│   ├── retrieval.py
│   ├── llm.py
│   ├── verification.py
│   ├── pipeline.py
│   ├── train.py
│   └── cli.py
├── tests/test_core.py
├── pyproject.toml
└── requirements.txt
```

## Installation

Python 3.10 or later is recommended.

```bash
git clone <repository-url>
cd mgr-kd
python -m venv .venv
```

Activate the environment and install the package:

```bash
pip install -e .
```

For development and testing:

```bash
pip install -e ".[dev]"
pytest
```

## Data preparation

Download FitKG-CN from [Zenodo](https://doi.org/10.5281/zenodo.14355004) and place the extracted files as follows:

```text
data/fitkg-cn/
├── train.json
├── dev.json
└── fitkg_types.json
```

The dataset is not redistributed by this repository. Its original license and terms remain applicable.

## Configuration

The default configuration is stored in `configs/default.yaml`. Important fields include the dataset path, embedding dimension, retrieval depth, LLM endpoint, and evidence-fusion weights.

MGR-KD uses an OpenAI-compatible local inference endpoint by default:

```yaml
llm:
  enabled: true
  base_url: http://127.0.0.1:1234/v1
  model: qwen/qwen3.5-9b
```

The LLM is constrained to relation labels and entity candidates supplied by the graph retriever. It cannot introduce new candidate entities.

## Training

```bash
mgr-kd train --config configs/default.yaml
```

The checkpoint is written to the configured `artifacts/` directory, which is excluded from version control.

## Knowledge discovery

Run a tail-entity discovery query with an entity name and a relation label:

```bash
mgr-kd discover \
  --config configs/default.yaml \
  --checkpoint artifacts/mgr_kd.pt \
  --head "示例头实体" \
  --relation "示例关系"
```

Disable LLM reasoning when only graph retrieval and verification are required:

```bash
mgr-kd discover \
  --config configs/default.yaml \
  --checkpoint artifacts/mgr_kd.pt \
  --head "示例头实体" \
  --relation "示例关系" \
  --no-llm
```

The command prints the final ranked candidates as JSON and does not write evaluation results into the repository.

## Python API

```python
from mgr_kd.config import load_config
from mgr_kd.pipeline import load_pipeline

config = load_config("configs/default.yaml")
pipeline = load_pipeline(config, "artifacts/mgr_kd.pt")
result = pipeline.discover(head_name="示例头实体", relation_name="示例关系")
```

## Reproducibility

- Random seeds are configured explicitly.
- Candidate generation is constrained by the observed relation schema.
- LLM decoding uses zero temperature.
- Invalid or unavailable LLM responses fall back to graph-derived scores.
- Runtime artifacts are isolated from tracked source files.

## Citation

Please cite the accompanying manuscript when using MGR-KD. Formal citation metadata can be added after publication.

## License

Copyright (c) 2026. No license for redistribution or derivative use is granted unless a separate license file is provided by the authors.
