# What is Drift?
## “When the world changes, but the model doesn’t.”
Drift is a change in the joint data distribution P(X,Y) over time.

--

# Types of Drift
## By what changes

<img src="figures/real_virtual_drift.png" alt="Drift types" style="width:70%">

Note:

Ας φανταστούμε ένα μοντέλο που ανιχνεύει απάτη σε συναλλαγές.
Αν αλλάξει μόνο ο τρόπος που μοιάζουν οι συναλλαγές — για παράδειγμα, περισσότερες online αγορές τα Χριστούγεννα — έχουμε virtual drift: αλλάζει η κατανομή των εισόδων, όχι η σχέση με την απάτη.
Αν όμως οι απατεώνες αλλάξουν στρατηγική και κάνουν μικρότερες, πιο “κανονικές” συναλλαγές, αλλάζει η σχέση μεταξύ εισόδου και εξόδου — αυτό είναι actual drift.
Και αν αλλάξουν και οι δύο πλευρές, δηλαδή και η συμπεριφορά των χρηστών και των απατεώνων, τότε έχουμε concept drift.

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

Algorithms assume that true labels are available in order for the model to be trained

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

# Block-based Methods
<!-- ## “Analyze the story in chapters” -->

Divides the data stream into blocks.
Each block is analyzed individually.

## How it works:

- In each block runs a change point algorithm

## Pros:

- Robust to noise
- Can detect multiple drifts within a block

## Cons:

- Slow reaction time (waits for block completion)
- Computationally heavy

Note:

# Change point algorithm

Looks for positions where the statistical distribution before and after changes significantly

Given a time-ordered sequence x_1, x_2,…,x_T, the algorithm tries to identify time 
indices τ_1 ,τ_2, …, τ_k where something fundamental in the data changes — like its 
mean, variance, correlation, or probability distribution

# Heavy

- Process large blocks of data at once,

- Often analyze the entire block holistically, not just sample summaries,

- And may search for multiple change points within the block

Examples: Kernel MMD, Wasserstein-based detectors

## Key distinction:

Block-based methods use large, mostly non-overlapping windows analyzed as independent data segments, not as continuously moving windows

--

# Batch-based Methods
<!-- ## “Compare yesterday and today” -->

They keep two smaller windows of data:
- A reference window (past concept)
- A detection window (recent data)

## How it works:

- Windows can be fixed, sliding, or adaptive
- When differences exceed a threshold $\rightarrow$ drift detected

## Pros:

- Detects both gradual and sudden drifts

## Cons:

- Choice of window size impacts sensitivity

Note:

They compare distributions (e.g., KS test, MMD) to check if the new batch differs significantly from the old one

Examples: NN-DVI, SQSI.

## Key distinction:

Batch-based methods rely on explicit comparison between two consecutive windows, focusing on contrast between past and present data.

--

# Online-based Methods
<!-- ## “React as the words change” -->

They detect drift continuously, updating statistics as each instance arrives

## How it works:

- Use sliding or adaptive window that is updated with every new data point

- Apply test statistics or hypothesis tests in real time

## Pros:

- Fastest detection, reacts immediately to change

## Cons:

- Sensitive to noise, prone to false alarms

Note:

1. Receive a new data point x_t

2. Update the current statistic (mean, variance, error, etc.) using x_t.

3. Compute a test statistic to check for significant change.

4. Trigger drift if the test statistic exceeds a threshold.

5. Reset or shrink the window if drift is detected.

# Test Statistic

A test statistic is simply a numerical measure of how much the current data deviates from what’s expected under the “no drift” assumption.

Then the detector checks: T_t > threshold⇒drift detected

- Mean-based statistics (Page-Hinkley)
- Variance-based statistics
- Distance-based statistics


Batch-based:

- Wait until you have 100 samples in the detection window.
- Compare it to the previous 100 samples (reference).
- Perform the test → decide drift or no drift.
- Slide the window (maybe by 10 or 50 samples) → repeat.

👉 Detection only happens after every batch, e.g., every 100 samples.

🔹 Online-based:

- As soon as each new sample arrives:
- Update your statistic (mean, error, likelihood, etc.).
- Check if it deviates significantly.
- If so → drift.

👉 Detection happens continuously, potentially every single data point.

# ADWIN

ADWIN (ADaptive WINdowing) maintains a dynamically sized window W of recent data.

1. When a new instance arrives, it adds it to W.
2. It splits W into two sub-windows W_1 and W_2.
3. It compares their means using a Hoeffding bound to test if they differ significantly.
4. If yes → drift detected, and old data (the earlier part of the window) is dropped.

Examples: ADWIN, DSDD, Page-Hinkley, SAND.

## Key distinction:

Online-based methods differ mainly in timing — they update continuously per instance, unlike batch or block methods that wait for windows to fill.

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


Note: 

- chi-square test for independence

- Hilbert–Schmidt Independence Criterion