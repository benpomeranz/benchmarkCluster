# Benchmark Clustering

Clusters AI model benchmarks by how similarly they rank models. Two benchmarks are "close" if a model's relative performance on one predicts its relative performance on the other.

## Data

50 benchmark CSV files covering 577 AI models, sourced from [Epoch AI's Benchmarking Hub](https://epochai.org/data/benchmarking). Data is licensed [CC-BY](https://creativecommons.org/licenses/by/4.0/) from Epoch AI.

## Usage

Open and run `cluster/more_improved_clustering.ipynb`. The notebook:

1. Loads all 50 benchmarks and filters to 27 with sufficient model overlap
2. Computes pairwise distances (mean |z-score difference| across shared models)
3. Clusters using hierarchical, KMeans, and spectral methods
4. Visualizes results via MDS projection and dendrograms

**To change the number of clusters:** modify `CHOSEN_K` in the last cell and re-run it.

See `cluster/results.md` for a full writeup of the methodology and findings.

### Dependencies

```
pip install pandas numpy matplotlib scipy scikit-learn
```
