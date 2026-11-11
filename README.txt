Router-Based Symbolic Reasoning Agent(RuleTaker / CounterBench / FOLIO)

Author : Gonuguntla Koushik
Roll no: M26CPS004


1. OVERVIEW
-----------
A reasoning agent that answers questions from three public benchmarks using
three symbolic engines behind one router:

  Engine A  RuleTaker     natural-language deduction
                          (forward chaining, negation as failure)
  Engine B  CounterBench  counterfactual causal reasoning
                          (structural causal model; abduction -> action ->
                          prediction by exact enumeration)
  Engine C  FOLIO         first-order logic entailment
                          (Horn forward chaining, then DPLL refutation)

Every engine has the same interface:

    engine.reason(context, query) ->
        {status, prediction, trace, error}

The router chooses the engine from the benchmark name of each split
(RuleTaker/, CounterBench/, FOLIO/).


2. REQUIREMENTS
---------------
  Python 3.9+ (Google Colab or Jupyter)
  Packages: datasets, huggingface_hub, pandas, tqdm, z3-solver
  Internet access to download the datasets from the Hugging Face Hub.

The first cell installs z3-solver:   !pip install -q z3-solver
Note: z3 is imported, but Engine C uses its own DPLL solver, not Z3.


3. HOW TO RUN THE NOTEBOOK
--------------------------
Run all cells TOP TO BOTTOM (order matters):

  1. Setup: install and imports
  2. Load datasets (7 splits from Hugging Face)
  3. Input preparation (common schema: engine, context, query, label, depth)
  4. Engine A (RuleTaker)
  5. Engine B (CounterBench)
  6. Engine C (FOLIO)
  7. Router
  8. Evaluation functions
  9. Run all seven splits (prints accuracy, coverage, status counts,
     accuracy by depth, and time per query)
 10. Label normalisation and submission.csv export

Datasets used (all public):
  RuleTaker     jise/ruletaker
                  (train 1000, test 250, OOD test 375)
  CounterBench  CounterBench/CounterBench
                  (data_balanced_alpha_V1.json: 1000,
                   data_balanced_backdoor_V2.json: 200)
  FOLIO         yale-nlp/FOLIO
                  (train 1001, validation 203)

The Hugging Face login in the notebook is OPTIONAL (the data is public).
It looks for a Colab secret; remove or rename the secret name in the
login cell before sharing the notebook publicly.

Known note: the last cell (submission.csv export) expects a pandas table
called "submission" with columns dataset_test_number, predicted, actual.
The step that builds this table is not in the saved notebook, so build it
from the evaluation results first (or skip that cell).


4. RESULTS (4,029 queries)
--------------------------
  Split                    n     Correct   Accuracy
  RuleTaker/train        1000      983      98.30 %
  RuleTaker/test          250      244      97.60 %
  RuleTaker/OOD           375      373      99.47 %
  CounterBench/alpha     1000      950      95.00 %
  CounterBench/backdoor   200      170      85.00 %
  FOLIO/train            1001      671      67.03 %
  FOLIO/validation        203      146      71.92 %
  ------------------------------------------------
  Overall                4029     3537      87.79 %

  Per engine: A 98.46 %, B 93.33 %, C 67.86 %
  Accuracy counts unanswered queries as wrong.
  Total run time is about 65 s (FOLIO takes most of it).
  Timings vary slightly between runs; accuracies were identical.

