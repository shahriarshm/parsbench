# Benchmarks

Using Benchmarks you can evaluate different Models based on different Tasks and compare their score.

## Custom Benchmark

You can easily create a benchmark with your desired tasks and models.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

from parsbench.benchmarks import CustomBenchmark
from parsbench.models import OpenAIModel, PreTrainedTransformerModel
from parsbench.tasks import ParsiNLUMultipleChoice, PersianMath, ParsiNLUReadingComprehension

# Create Models
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2-72B-Instruct",
    torch_dtype="auto",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2-72B-Instruct")
qwen2_model = PreTrainedTransformerModel(model=model, tokenizer=tokenizer)

aya_model = OpenAIModel(
    api_base_url="http://localhost:11434/v1/",
    api_secret_key="ollama",
    model="aya:latest",
)

# Run Benchmark
benchmark = CustomBenchmark(
    models=[qwen2_model, aya_model],
    tasks=[
        ParsiNLUMultipleChoice,
        ParsiNLUReadingComprehension,
        PersianMath,
    ],
)
result = benchmark.run(
    prompt_lang="fa",
    prompt_shots=[0, 3],
    n_first=100,
    sort_by_score=True,
)
```

## Full Benchmark

To benchmark your model based on all existing tasks in the framework. You can use `load_all_tasks` function.

```python
from parsbench.benchmarks import CustomBenchmark
from parsbench.models import OpenAIModel
from parsbench.tasks.utils import load_all_tasks

aya_model = OpenAIModel(
    api_base_url="http://localhost:11434/v1/",
    api_secret_key="ollama",
    model="aya:latest",
)

# Run Benchmark
benchmark = CustomBenchmark(
    models=[aya_model],
    tasks=load_all_tasks(),
)
result = benchmark.run(
    prompt_lang="fa",
    prompt_shots=[0, 3],
    n_first=100,
    sort_by_score=True,
)
```

## Benchmark Result

The benchmark result contains all evaluation results for each model. You can use it directly or convert it to
a Pandas DataFrame with `to_pandas` function. If you want to get a pivot table of benchmark result, you should use `to_pandas(pivot=True)`.

```python
print(result.to_pandas(pivot=True))
```

Output should be like:

```txt
                                                                                     score          
model_name                                                                     qwen2:latest          
n_shots                                                                                   0         3
task_category task_name                      sub_task         score_name                             
classic       ParsiNLU Reading Comprehension NaN              Common Tokens         0.46231  0.588274
knowledge     PersiNLU Multiple Choice       common_knowledge Exact Match           0.30000  0.000000
                                             literature       Exact Match           0.20000  0.428571
                                             math_and_logic   Exact Match           0.60000  0.285714
math          Persian Math                   NaN              Math Equivalence      0.00000  0.142857
```

Note: It would look better if you run it in a Jupyter Notebook.

### Radar Plot (Spider Plot)

For a better comparison between models performance on different tasks. You can use `show_radar_plot` to visualize the benchmark.

```python
result.show_radar_plot()
```

Output should be like:

![Benchmark Bar Plot](../imgs/radarplot.png)

### Save Result

To save the matches, evaluations and benchmark results during the benchmarking process, you can set `save_matches`, `save_evaluation` and `save_benchmark`.

```python
benchmark = CustomBenchmark(
    models=[aya_model, qwen2_model],
    tasks=[PersianMath, FarsTailEntailment],
)
result = benchmark.run(
    prompt_lang="fa",
    prompt_shots=[0, 5],
    n_first=100,
    save_matches=True,
    save_evaluation=True,
    save_benchmark=True,
    output_path="results",
    sort_by_score=True,
)
```

The output directory structure should be like this:

```txt
results
├── aya:latest
│   ├── FarsTail_Entailment
│   │   ├── evaluation.jsonl
│   │   ├── matches_0_shot.jsonl
│   │   └── matches_5_shot.jsonl
│   └── Persian_Math
│       ├── evaluation.jsonl
│       ├── matches_0_shot.jsonl
│       └── matches_5_shot.jsonl
├── qwen2:latest
│   ├── FarsTail_Entailment
│   │   ├── evaluation.jsonl
│   │   ├── matches_0_shot.jsonl
│   │   └── matches_5_shot.jsonl
│   └── Persian_Math
│       ├── evaluation.jsonl
│       ├── matches_0_shot.jsonl
│       └── matches_5_shot.jsonl
└── benchmark.jsonl
```
