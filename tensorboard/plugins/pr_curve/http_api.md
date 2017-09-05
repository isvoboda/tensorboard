# Precision—Recall Curve Plugin HTTP API

The plugin name is `pr_curves`, so all its routes are under
`/data/plugin/pr_curves`.

## `/data/plugin/pr_curves/available_time_entries`

Retrieves a JSON object mapping run name to a list of time entries (one for each
step). Each time entry has 2 properties:

* **step**: The step of the event.
* **wall_time**: The time in seconds since the epoch at which the summary was
  written.

Here is an example.

```json
{
  "foo": [
    {
      "step": 0,
      "wall_time": 1503076940.949388,
    },
    {
      "step": 1,
      "wall_time": 1503076940.953447,
    },
    {
      "step": 2,
      "wall_time": 1503076940.95812,
    }
  ],
  "bar": [
    {
      "step": 0,
      "wall_time": 1503076940.964225,
    },
    {
      "step": 1,
      "wall_time": 1503076940.969845,
    },
    {
      "step": 2,
      "wall_time": 1503076940.974917,
    }
  ],
}
```

## `/data/plugin/pr_curves/pr_curves`

Retrieves a JSON object mapping run name to a list of PR curve data entries (one
entry per step). This route requires a `tag` GET parameter as well as at least
one `run` GET parameter to specify both the tag and list of runs to retrieve
data for. 

Each PR data entry contains the following properties.

* **wall_time**: The wall time (number) in seconds since the epoch at which data
  for the PR curve was collected.
* **step**: The step (number).
* **precision**: A list of precision values (numbers). The length of this list
  is the number of thresholds used to generate PR curves.
* **recall**: A list of recall values that each pair-wise correspond to the list
  of precision values (numbers).
* **true_positives**: A list of true positive counts (ints).
* **false_positives**: A list of false positive counts (ints).
* **true_negatives**: A list of true negative counts (ints).
* **false_negatives**: A list of false negative counts (ints).
* **thresholds**: A list of increasing thresholds (floats from 0 to 1).

Here is an example. We assume 5 thresholds for PR curves. We also assume GET
parameters of `?tag=green/pr_curves&run=bar&run=foo`.

```json
{
  "bar": [
    {
      "wall_time": 1503076940.949388,
      "step": 0,
      "precision": [0.3333333, 0.3853211, 0.5421686, 0.75, 1.0],
      "recall": [1.0, 0.8400000, 0.3000000, 0.0400000, 0.0],
      "true_positives": [150, 126, 45, 6, 0],
      "false_positives": [300, 201, 38, 2, 0],
      "true_negatives": [0, 99, 262, 298, 300],
      "false_negatives": [0, 24, 105, 144, 150],
      "thresholds": [0.0, 0.25, 0.5, 0.75, 1.0],
    },
    {
      "wall_time": 1503076940.953447,
      "step": 1,
      "precision": [0.3333333, 0.3855421, 0.5357142, 0.4000000, 1.0],
      "recall": [1.0, 0.8533333, 0.3000000, 0.0266666, 0.0],
      "true_positives": [150, 128, 45, 4, 0],
      "false_positives": [300, 204, 39, 6, 0],
      "true_negatives": [0, 96, 261, 294, 300],
      "false_negatives": [0, 22, 105, 146, 150],
      "thresholds": [0.0, 0.25, 0.5, 0.75, 1.0],
    },
  ],
  "foo": [
    {
      "wall_time": 1503076940.964225,
      "step": 0,
      "precision": [0.3333333, 0.3786982, 0.5384616, 1.0, 1.0],
      "recall": [1.0, 0.8533334, 0.28, 0.0666667, 0.0],
      "true_positives": [75, 64, 21, 5, 0],
      "false_positives": [150, 105, 18, 0, 0],
      "true_negatives": [0, 45, 132, 150, 150],
      "false_negatives": [0, 11, 54, 70, 75],
      "thresholds": [0.0, 0.25, 0.5, 0.75, 1.0],
    },
    {
      "wall_time": 1503076940.969845,
      "step": 1,
      "precision": [0.3333333, 0.3850932, 0.5, 0.25, 1.0],
      "recall": [1.0, 0.8266667, 0.28, 0.0133333, 0.0],
      "true_positives": [75, 62, 21, 1, 0],
      "false_positives": [150, 99, 21, 3, 0],
      "true_negatives": [0, 51, 129, 147, 150],
      "false_negatives": [0, 13, 54, 74, 75],
      "thresholds": [0.0, 0.25, 0.5, 0.75, 1.0],
    },
  ],
}
```

Used by the PR Curves dashboard to render plots.

## `/data/plugin/pr_curves/tags`

Retrieves a JSON object whose keys are the names of all the runs (regardless of
whether they have available PR curve data). The values of that object are also
objects whose keys are tag strings (associated with the run) and whose values
are objects with 2 keys: `displayName` and `description` (associated with the
run-tag combination).

The `displayName` is shown atop individual plots in TensorBoard. The description
might offer insight for instance into how data was generated.

Importantly, the `description` contains sanitized HTML to be injected into the
DOM, while the `displayName` is simply an arbitrary string.

Here is an example.

```json
{
  "foo": {
    "green/pr_curves": {
      "displayName": "classifying green",
      "description": "Human eyes are very sensitive to green."
    },
    "red/pr_curves":  {
      "displayName": "classifying red",
      "description": "Human eyes are also pretty sensitive to red."
    },
  },
  "bar": {
    "green/pr_curves": {
      "displayName": "classifying green",
      "description": "Human eyes are very sensitive to green."
    },
    "blue/pr_curves":  {
      "displayName": "classifying blue",
      "description": "Human eyes are not as sensitive to blue."
    },
  },
}
```

Used by TensorBoard for categorizing PR Curve plots into runs and tags as well
as to show metadata associated with each plot.
