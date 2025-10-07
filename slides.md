# What is Drift?
## “When the world changes, but the model doesn’t.”
Drift is a change in the joint data distribution P(X,Y) over time.

--

# Some Real world examples on where there is drift maybe?

--

# Types of Drift
## By what changes

<img src="figures/real_virtual_drift.png" alt="Drift types" style="width:70%">

--

# Types of Drift
## By how it changes

<img src="figures/drift_classes.png" alt="Drift types_in_time" style="width:70%">

--

# Why do we Detect Drift?
- Detect changes in data or concept $\rightarrow$ model adaptation (e.g. retraining)
- Ensure the model remains accurate and relevant as the world changes

---

# How do we Detect Drift?

Process called drift detection which monitors a system over time to spot changes in its behavior or performance.

Note: It helps identify when things start to deviate from expected standards—like a model becoming less accurate—so that adjustments can be made to keep quality high.

--

# Supervised Drift Detection

Algorithms assume that true labels are available for at least some portion of the incoming data in order for the model to be trained

--

# Semi-supervised Drift Detection

These algorithms use **both features and labels** when labels are **partially available**

Note:

Labels are often delayed, sparse, or costly to obtain

--

# Unsupervised Drift Detection

Unsupervised drift detection algorithms rely **only on the input features** $X$

---

# Drift Detection Categories

--

# Block-based Methods — “Analyze the story in chapters”

They divide the data stream into large, fixed-size blocks.
Each block is treated as a complete statistical unit, analyzed individually or compared with previous blocks.

## How it works:

- Each block represents a full distribution snapshot.

- Drift can be detected within a block or between consecutive blocks.

## Pros:

- Robust to noise.

- Can detect multiple drifts within a block.

## Cons:

- Slow reaction time (waits for block completion).

- Computationally heavier.

Note:

Examples: Kernel MMD, Wasserstein-based detectors.

## Key distinction:

Block-based methods use large, mostly non-overlapping windows analyzed as independent data segments, not as continuously moving windows.

--

# Batch-based Methods — “Compare yesterday and today”

They keep two smaller windows of data:
- A reference window (past concept).
- A detection window (recent data).

They compare distributions (e.g., KS test) to check if the new batch differs significantly from the old one

## How it works:

- Windows can be fixed, sliding, or adaptive.
- When differences exceed a threshold $\rightarrow$ drift detected.

## Pros:

- Detects both gradual and sudden drifts.

## Cons:

- Must wait for both windows to fill.
- Choice of window size impacts sensitivity.

Note:

Examples: NN-DVI, SQSI.

## Key distinction:

Batch-based methods rely on explicit comparison between two consecutive windows, focusing on contrast between past and present data.

--

# Online-based Methods — “React as the words change”

They detect drift continuously, updating statistics as each instance arrives.

## How it works:

- Use sliding or adaptive windows that move forward with every new data point.

- Apply incremental statistics or hypothesis tests in real time.

## Pros:

- Fastest detection, reacts immediately to change.

## Cons:

- Sensitive to noise, prone to false alarms

Note:

Examples: ADWIN, DSDD, Page-Hinkley, SAND.

## Key distinction:

Online-based methods differ mainly in timing — they update continuously per instance, unlike batch or block methods that wait for windows to fill.

--

# Meta-statistic-based Methods — “Watch the watcher”

They don’t analyze raw data directly — they monitor how a model behaves.

## How it works:

- Monitors metrics such as error rate or loss

- Apply a statistical test (e.g., ADWIN) to detect significant changes in these metrics.

## Pros:

- Model-agnostic, works with any learner.

- Can function even without labels (unsupervised drift).

## Cons:

- Indirect, relies on model accuracy which may miss subtle (virtual) drifts

Note:

Model-agnostic means:

1. Doesn’t assume a specific data distribution,

2. Compares windows of data, and

3. Tests for statistical independence between features and labels (or between time and data)

Examples: Model + ADWIN, loss-based drift detection, reconstruction error drift detection.

## Key distinction:

Meta-statistic methods differ in what they monitor — they watch model performance or error trends, not the raw input data distributions.

---

# Dynamic Adapting Window Independence Drift Detection (DAWIDD)

Turns concept drift into an independence test

Note:

The whole philosophy behind DAWIDD is very simple but powerful:
if the data distribution does not change over time, then the data 𝑋 and time 𝑇 should be independent.
But if the data distribution does change — meaning a concept drift has occurred — then 𝑋 and 𝑇 become dependent.

--

# 1. Attach Time to Each Sample

Transform our data stream into a sequence of pairs, each data point is now represented as ($x_i$, $t_i$), where $x_i$ is the feature vector and $t_i$ is the timestamp

--

# 2. Maintain a Dynamically Adapting Window

This window, $W$, is constantly updated:

- Each time a new data point arrives, it’s added to the window

- The window size is controlled by two parameters:

  - $n_{min}$: the minimum number of samples needed before we can reliably run a statistical test

  - $n_{max}$: the maximum number of samples allowed before we start dropping old ones.

Once the window exceeds $n_{max}$, we randomly delete some points rather than just discarding the oldest.

--

# 3. Testing Independence Between X and T

Once we have enough samples in our window (at least $n_{min}$), we perform a statistical test to check whether the data values $X$ are independent of the time values $T$.

- If the $H_0$ holds, meaning $X$ and $T$ are independent, there’s **no** drift
  - Keep adding **new** samples to the window

- If $H_0$ is reject, that means the distribution of $X$ has changed with time. So, a drift is detected
  - We signal a **drift event**
  - **Shrink** the window down to a small size, typically keeping only the last $n_{min}$ samples