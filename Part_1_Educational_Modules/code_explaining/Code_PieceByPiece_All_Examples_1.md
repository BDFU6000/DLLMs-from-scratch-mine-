# 📖 Piece-by-Piece Code Explanations — Part 1 (All Examples)

> **What this is:** every code example in the module (§1–§8), explained the same way — the **actual code** shown, then a **line-by-line** plain-English walkthrough of what each line does, what each argument means, and how it links to the concepts. Read it top to bottom or jump to any file.
>
> *(Sections 9–19 are concept-only and ship no code, so they aren't here.)*

**How to read a line explanation:** each bullet quotes a line of code, then explains it in plain words — exactly like a teacher narrating over your shoulder.

---

# §1 · Foundations

## `math_basics.py`

### `linear_algebra_examples()`
```python
scalar = 5
vector = np.array([1, 2, 3])
matrix = np.array([[1, 2], [3, 4], [5, 6]])
tensor = np.random.rand(3, 224, 224)
dot_product = np.dot(matrix_a, matrix_b)
eigenvalues, eigenvectors = np.linalg.eig(square_matrix)
```
- **`scalar = 5`** — a single number (0 dimensions). The simplest data container.
- **`vector = np.array([1, 2, 3])`** — a 1-D list of numbers. Think "a single row of data."
- **`matrix = np.array([[1,2],[3,4],[5,6]])`** — a 2-D grid (3 rows × 2 columns). A spreadsheet of numbers.
- **`tensor = np.random.rand(3, 224, 224)`** — a 3-D block of random numbers. This exact shape is an RGB image: 3 colour channels, each 224×224 pixels. `rand` fills it with random values between 0 and 1.
- **`np.dot(matrix_a, matrix_b)`** — **matrix multiplication**. This single operation is what a neuron does when it computes `Σ(weights × inputs)`; it's the most-used calculation in all of deep learning.
- **`np.linalg.eig(square_matrix)`** — computes **eigenvalues and eigenvectors**. You don't need the math now; just know this is the engine inside PCA (the dimensionality-reduction algorithm from §2).

### `calculus_examples()`
```python
def f(x): return x**2
x_val, h = 3.0, 1e-5
numerical_derivative = (f(x_val + h) - f(x_val)) / h
exact_derivative = 2 * x_val
```
- **`def f(x): return x**2`** — defines a simple function, f(x) = x².
- **`x_val, h = 3.0, 1e-5`** — the point we want the slope at (`x=3`) and a tiny step `h` (0.00001).
- **`(f(x_val + h) - f(x_val)) / h`** — the **definition of a derivative**: "how much does the output change when I nudge the input a tiny bit?" It returns ≈ 6.
- **`exact_derivative = 2 * x_val`** — the textbook answer (the derivative of x² is 2x, so 2×3 = 6). The two match, proving the numerical method works. **Why it matters:** this slope *is* the gradient, and doing this across a whole network is **backpropagation** — how models learn (§3).

### `probability_examples()`
```python
normal_samples = np.random.normal(loc=0.0, scale=1.0, size=1000)
bernoulli_samples = (np.random.uniform(0, 1, 1000) < 0.7).astype(int)
```
- **`np.random.normal(loc=0.0, scale=1.0, size=1000)`** — draws 1000 random numbers from the **Normal (bell-curve) distribution**, centred at `loc=0` with spread `scale=1`. This is how neural-network weights are randomly initialised before training.
- **`(np.random.uniform(0,1,1000) < 0.7).astype(int)`** — simulates a **Bernoulli** (yes/no) trial with 70% chance of "1": roll a random number 0–1, mark it `1` if below 0.7, else `0`. This is literally how the synthetic churn labels were generated.

---

## `numpy_pandas_basics.py`

### `numpy_examples()`
```python
arr = np.array([1, 2, 3, 4, 5])
squared_arr = arr ** 2
matrix = np.array([[1, 2, 3], [4, 5, 6]])
matrix_plus_10 = matrix + 10
np.sum(matrix); np.mean(matrix, axis=0)
```
- **`arr ** 2`** — **vectorization**: squares all five numbers at once, no loop. This is why NumPy/PyTorch are fast enough for ML.
- **`matrix + 10`** — **broadcasting**: the single `10` is automatically applied to every cell. This is exactly how a bias gets added across a whole layer of a network.
- **`np.sum(matrix)`** — adds up every number in the grid.
- **`np.mean(matrix, axis=0)`** — averages **down the columns** (`axis=0` = collapse the rows). `axis=1` would average across each row instead.

### `pandas_examples()`
```python
df = pd.DataFrame({'Age':[...], 'Salary':[...], 'Department':[...]})
df['Salary']
df[df['Age'] > 30]
df.groupby('Department')['Salary'].mean()
df_missing['Salary'].fillna(df_missing['Salary'].mean())
```
- **`pd.DataFrame({...})`** — builds a table from a dictionary; each key becomes a column. This is the "Excel sheet" you work with.
- **`df['Salary']`** — selects one column by name.
- **`df[df['Age'] > 30]`** — **boolean filtering**: the inside makes a True/False mask, and the outside keeps only the `True` rows (everyone older than 30).
- **`df.groupby('Department')['Salary'].mean()`** — **split-apply-combine**: group rows by Department, then average each group's Salary. This is the exact move behind "churn rate by contract type."
- **`fillna(df['Salary'].mean())`** — **cleaning missing data**: replace blank (`NaN`) salaries with the column's average so the model doesn't choke on holes.

---

# §2 · Machine Learning

## `ml_algorithms.py`

### `linear_regression_example()`  — the full template
```python
X, y = make_regression(n_samples=100, n_features=1, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
model = LinearRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_test)
mse = mean_squared_error(y_test, predictions)
model.coef_[0]      # slope m
model.intercept_    # bias b
```
- **`make_regression(n_samples=100, n_features=1, noise=10, random_state=42)`** — invents a fake dataset: `100` points, each with `1` input feature, with `noise=10` scattering them realistically. `random_state=42` freezes the randomness so you get identical data every run. Returns inputs `X` (shape `(100,1)`) and answers `y`. Because `y` comes *with* `X`, this is **supervised learning**.
- **`train_test_split(..., test_size=0.2, ...)`** — splits into 80 training points and 20 test points held back. You train on some and test on the unseen rest to check it **generalizes** rather than memorizes.
- **`model = LinearRegression()`** — creates the empty model; it knows the algorithm but hasn't learned yet.
- **`model.fit(X_train, y_train)`** — the **learning** step: finds the straight line (slope `m`, intercept `b`) closest to the training points by minimizing squared error. This is `y = mx + b`.
- **`model.predict(X_test)`** — uses that learned line to guess `y` for the 20 unseen test inputs.
- **`mean_squared_error(y_test, predictions)`** — scores the guesses: average of the squared gaps between prediction and truth. Lower = better.
- **`model.coef_[0]`** — the learned **slope m** (`coef_` has one per feature; `[0]` is the only one).
- **`model.intercept_`** — the learned **bias b** (where the line crosses the y-axis).

### `classification_examples()`
```python
X, y = make_classification(n_samples=200, n_features=4, n_classes=2, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
LogisticRegression().fit(X_train, y_train)
RandomForestClassifier(n_estimators=50, random_state=42).fit(X_train, y_train)
SVC(kernel='linear').fit(X_train, y_train)
accuracy_score(y_test, lr_model.predict(X_test))
```
- **`make_classification(n_samples=200, n_features=4, n_classes=2, ...)`** — fake **labeled** data: 200 points, 4 features each, 2 categories. Same supervised setup, but now we predict a *category* not a number.
- **`train_test_split(...)`** — same 80/20 hold-out as before.
- **`LogisticRegression().fit(...)`** — trains the S-curve classifier (the model behind your spam filter). `.fit` learns the boundary between the two classes.
- **`RandomForestClassifier(n_estimators=50, ...)`** — trains **50 decision trees** (`n_estimators=50`) that vote together; the tabular champion from your churn project.
- **`SVC(kernel='linear')`** — a Support Vector Machine finding the widest-margin straight boundary (`kernel='linear'` = straight line).
- **`accuracy_score(y_test, ...predict...)`** — fraction of test points classified correctly. Fine here because the data is balanced; on imbalanced data you'd switch to precision/recall (the "accuracy trap").

### `unsupervised_examples()`
```python
X_blobs, _ = make_blobs(n_samples=150, centers=3, n_features=2, random_state=42)
KMeans(n_clusters=3, random_state=42, n_init=10).fit(X_blobs)   # .cluster_centers_
pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X_high_dim)                       # .explained_variance_ratio_
```
- **`make_blobs(n_samples=150, centers=3, n_features=2, ...)`** — makes 150 points naturally clumped into 3 blobs. Note the labels are thrown away (`_`) — this is **unsupervised**.
- **`KMeans(n_clusters=3, n_init=10).fit(X_blobs)`** — finds 3 cluster centres. `n_clusters=3` = how many groups to look for; `n_init=10` = try 10 random starts and keep the best. Afterwards `.cluster_centers_` holds the 3 centroids it discovered.
- **`PCA(n_components=2)`** — set up dimensionality reduction down to 2 dimensions.
- **`pca.fit_transform(X_high_dim)`** — squeezes the 10-feature data into 2 features in one call. `.explained_variance_ratio_` then tells you how much of the original information those 2 dimensions kept.

---

# §3 · Deep Learning

## `basic_neural_network.py`
```python
class SimpleFeedForwardNN(nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        super().__init__()
        self.layer1 = nn.Linear(input_size, hidden_size)
        self.activation = nn.GELU()
        self.layer2 = nn.Linear(hidden_size, num_classes)
    def forward(self, x):
        out = self.layer1(x); out = self.activation(out); out = self.layer2(out)
        return out
```
- **`class SimpleFeedForwardNN(nn.Module)`** — defines a neural network. Every PyTorch model inherits from `nn.Module` (which handles the bookkeeping of parameters).
- **`super().__init__()`** — required boilerplate that initialises the parent `nn.Module`.
- **`self.layer1 = nn.Linear(input_size, hidden_size)`** — a layer of neurons that computes `z = Wx + b`. The weights `W` and biases `b` inside it **are the parameters** that learning will tune.
- **`self.activation = nn.GELU()`** — the non-linear "bend" (the activation function GPT and BERT use).
- **`self.layer2 = nn.Linear(hidden_size, num_classes)`** — the output layer, producing one score per class.
- **`def forward(self, x)`** — defines the **forward pass**: how data flows through the layers. `layer1 → GELU → layer2`. Calling `model(x)` runs this.

```python
model = SimpleFeedForwardNN(input_size, hidden_size, num_classes)
criterion = nn.CrossEntropyLoss()
optimizer = optim.AdamW(model.parameters(), lr=learning_rate)
for epoch in range(epochs):
    predictions = model(dummy_inputs)
    loss = criterion(predictions, dummy_targets)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```
- **`model = SimpleFeedForwardNN(...)`** — actually build the network with chosen sizes.
- **`criterion = nn.CrossEntropyLoss()`** — the **loss function** for classification (the same one LLMs use for next-token prediction). It measures how wrong the predictions are.
- **`optimizer = optim.AdamW(model.parameters(), lr=learning_rate)`** — the **optimizer**: the engine that adjusts every parameter. `model.parameters()` hands it all the weights to tune; `lr` is the step size. AdamW's built-in weight decay = the **L2 regularization** from §2.
- **`for epoch in range(epochs)`** — repeat the whole training cycle `epochs` times.
- **`predictions = model(dummy_inputs)`** — **forward pass**: run the data through to get outputs.
- **`loss = criterion(predictions, dummy_targets)`** — **measure the error** against the true labels.
- **`optimizer.zero_grad()`** — wipe the gradients left over from the previous step (PyTorch adds them up otherwise).
- **`loss.backward()`** — **backpropagation**: compute the gradient (slope) for every weight, telling each which way to move.
- **`optimizer.step()`** — take one downhill step: nudge every weight in the direction that lowers the loss. Over many epochs, the loss drops and the model gets smarter.

---

# §4 · Neural Network Architectures

## `cnn_rnn_basics.py`

### `SimpleCNN` (for images)
```python
self.conv1 = nn.Conv2d(in_channels=1, out_channels=16, kernel_size=3, stride=1, padding=1)
self.relu = nn.ReLU()
self.pool = nn.MaxPool2d(kernel_size=2, stride=2)
self.fc = nn.Linear(16 * 14 * 14, num_classes)

def forward(self, x):
    x = self.conv1(x); x = self.relu(x); x = self.pool(x)
    x = torch.flatten(x, 1)
    x = self.fc(x)
    return x
```
- **`nn.Conv2d(in_channels=1, out_channels=16, kernel_size=3, stride=1, padding=1)`** — the convolution layer. `in_channels=1` = grayscale input (1 colour channel); `out_channels=16` = use 16 different filters (so it learns 16 patterns); `kernel_size=3` = each filter is 3×3; `stride=1` = slide one pixel at a time; `padding=1` = add a 1-pixel border so edges are handled and the size is preserved.
- **`nn.ReLU()`** — the activation applied after the convolution (keeps positives, zeroes negatives).
- **`nn.MaxPool2d(kernel_size=2, stride=2)`** — pooling: slide a 2×2 window keeping only the max, which **halves** width and height (e.g. 28×28 → 14×14).
- **`nn.Linear(16 * 14 * 14, num_classes)`** — the final classifier. `16*14*14` is the flattened size after one conv + pool on a 28×28 image (16 feature maps of 14×14).
- **`x = self.conv1(x); x = self.relu(x); x = self.pool(x)`** — the forward flow: detect patterns → activate → shrink.
- **`torch.flatten(x, 1)`** — flatten the 3-D feature maps into a 1-D vector per image (the `1` means "keep the batch dimension, flatten the rest") so the `Linear` layer can read it. This is the "Flatten → Fully Connected" step.
- **`x = self.fc(x)`** — produce the final class scores.

### `SimpleLSTM` (for sequences/text)
```python
self.lstm = nn.LSTM(input_size=300, hidden_size=128, batch_first=True)
self.fc = nn.Linear(hidden_size, num_classes)

def forward(self, x):
    lstm_out, (h_n, c_n) = self.lstm(x)
    last_hidden_state = h_n[-1]
    out = self.fc(last_hidden_state)
    return out
```
- **`nn.LSTM(input_size=300, hidden_size=128, batch_first=True)`** — the recurrent layer. `input_size=300` = each word is a 300-number embedding; `hidden_size=128` = the memory vector is 128 numbers wide; `batch_first=True` = expect input shaped `(batch, sequence, features)`.
- **`nn.Linear(hidden_size, num_classes)`** — turns the final memory into class scores.
- **`lstm_out, (h_n, c_n) = self.lstm(x)`** — running the LSTM returns: `lstm_out` (the output at every word), plus `h_n` (the **final hidden state** = the accumulated memory) and `c_n` (the cell state). The `h_n` is the "France→French" memory.
- **`last_hidden_state = h_n[-1]`** — grab the last layer's final memory — a summary of the whole sentence.
- **`out = self.fc(last_hidden_state)`** — classify that summary (e.g. positive vs negative sentiment).

---

## `transformer_attention.py` ⭐
```python
def scaled_dot_product_attention(query, key, value, mask=None):
    d_k = query.size(-1)
    scores = torch.matmul(query, key.transpose(-2, -1)) / math.sqrt(d_k)
    if mask is not None:
        scores = scores.masked_fill(mask == 0, -1e9)
    attention_weights = F.softmax(scores, dim=-1)
    output = torch.matmul(attention_weights, value)
    return output, attention_weights
```
This one function is the literal formula `softmax(Q·Kᵀ / √dₖ) · V`.
- **`def ...(query, key, value, mask=None)`** — takes the three attention inputs (Query, Key, Value) and an optional mask. Q = "what each word is looking for," K = "what each word advertises," V = "the info each word carries."
- **`d_k = query.size(-1)`** — the length of each Query/Key vector (the last dimension). Needed for the scaling step.
- **`torch.matmul(query, key.transpose(-2, -1))`** — **Q · Kᵀ**: dot every Query with every Key, producing a `seq × seq` grid of match scores (how strongly word *i* should attend to word *j*). `key.transpose(-2,-1)` flips the Key matrix so the shapes line up for multiplication.
- **`/ math.sqrt(d_k)`** — **scaling**: divide by √(vector size) to stop the scores from getting huge, which would make softmax too extreme and gradients unstable.
- **`if mask is not None: scores.masked_fill(mask == 0, -1e9)`** — the **causal mask**: wherever the mask is 0 (future positions), set the score to ≈ −∞, so after softmax those become 0. This enforces "a word can't see the future" in decoder/GPT models.
- **`F.softmax(scores, dim=-1)`** — convert each row of scores into **attention weights** that sum to 1 (a probability distribution over which words to focus on).
- **`torch.matmul(attention_weights, value)`** — blend the **Values** using those weights → each word's new, context-aware representation (the "bank → money/river" effect).
- **`return output, attention_weights`** — return the result plus the weights (the weights are exactly what an attention heatmap visualises).

---

# §5 · Large Language Models

## `llm_generation_example.py`
```python
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
model = GPT2LMHeadModel.from_pretrained("gpt2")
model.eval()
input_ids = tokenizer.encode("The future of artificial intelligence is", return_tensors="pt")
with torch.no_grad():
    output_ids = model.generate(input_ids, max_length=50, do_sample=True,
                                temperature=0.7, top_k=50, no_repeat_ngram_size=2,
                                pad_token_id=tokenizer.eos_token_id)
generated_text = tokenizer.decode(output_ids[0], skip_special_tokens=True)
```
- **`GPT2Tokenizer.from_pretrained("gpt2")`** — downloads GPT-2's tokenizer, the tool that turns text into token IDs and back.
- **`GPT2LMHeadModel.from_pretrained("gpt2")`** — downloads the **pre-trained** GPT-2 model (a decoder-only Transformer). "Pre-trained" means Phase-1 pretraining is already done for you.
- **`model.eval()`** — switch to inference mode (turns off training-only behaviours like dropout).
- **`tokenizer.encode(prompt, return_tensors="pt")`** — turn the prompt into a tensor of token IDs (`"pt"` = PyTorch format).
- **`with torch.no_grad():`** — turn off gradient tracking; we're generating, not training, so this is faster and lighter.
- **`model.generate(...)`** — runs the **auto-regressive loop**: predict the next token, append it, predict again, until done. The knobs:
  - **`max_length=50`** — stop after 50 tokens total (the context budget).
  - **`do_sample=True`** — pick the next token *probabilistically* instead of always the single most likely one (makes output less robotic).
  - **`temperature=0.7`** — the randomness dial: lower = safer/repetitive, higher = more creative/risky.
  - **`top_k=50`** — only ever sample from the 50 most likely next tokens (filters out nonsense).
  - **`no_repeat_ngram_size=2`** — forbid repeating any 2-word phrase (stops loops).
  - **`pad_token_id=tokenizer.eos_token_id`** — housekeeping so the generator knows what to pad with.
- **`tokenizer.decode(output_ids[0], skip_special_tokens=True)`** — turn the generated token IDs back into readable text (`skip_special_tokens` hides internal markers).

> **Insight:** this GPT-2 is a **base model**, so it just *continues* your prompt — it hasn't had the SFT + RLHF phases that make a chatbot. The "base model isn't an assistant" idea, live in code.

---

# §6 · Tokenization

## `bpe_example.py`
```python
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
text = "The player is playing unbelievablyfast."
token_ids = tokenizer.encode(text)
tokens = [tokenizer.decode([token_id]) for token_id in token_ids]
vocab_size = tokenizer.vocab_size
weird_tokens = tokenizer.encode("Hello 🚀 xyz")
```
- **`GPT2Tokenizer.from_pretrained("gpt2")`** — loads GPT-2's **Byte Pair Encoding (BPE)** tokenizer, already trained on a huge text corpus so it knows which subword chunks are common.
- **`text = "The player is playing unbelievablyfast."`** — the sentence to tokenize. `unbelievablyfast` is a made-up word on purpose, to show how BPE copes with words it's never seen.
- **`token_ids = tokenizer.encode(text)`** — converts the whole sentence into a list of integer **token IDs** (the numbers the model actually reads).
- **`tokens = [tokenizer.decode([token_id]) for token_id in token_ids]`** — a list comprehension that decodes **each ID back to its text piece individually**, so you can *see* how the sentence was chopped up. The fake word gets split into known chunks like `un`, `bel`, `iev`, `ably`, `fast` — proving BPE never needs a word in its vocabulary; it just assembles known pieces.
- **`vocab_size = tokenizer.vocab_size`** — how many distinct tokens GPT-2 knows (~50,257). Small enough to be efficient, big enough to cover language.
- **`tokenizer.encode("Hello 🚀 xyz")`** — shows BPE handling an emoji and gibberish: it falls back to byte-level pieces, so **nothing is ever truly "unknown."** That robustness is BPE's superpower.

> **Link to the markdown:** this is the "subwords, not whole words" idea made concrete — `1 token ≈ 0.75 words`, a fixed vocabulary, and graceful handling of unseen text.

---

# §7 · Model Training Details

## `training_loop.py`
```python
model = nn.Sequential(
    nn.Linear(input_size, 64),
    nn.ReLU(),
    nn.Linear(64, output_size)
)
criterion = nn.CrossEntropyLoss()
optimizer = optim.AdamW(model.parameters(), lr=learning_rate)

for epoch in range(epochs):
    model.train()
    optimizer.zero_grad()
    predictions = model(X_train)
    loss = criterion(predictions, y_train)
    loss.backward()
    optimizer.step()
    print(f"Epoch [{epoch+1}/{epochs}] - Loss: {loss.item():.4f}")
```
This is the §3 training loop again, but written with `nn.Sequential` and printing the loss so you can watch it learn.
- **`nn.Sequential(...)`** — a quick way to stack layers in order without writing a class. Here: `Linear(input→64) → ReLU → Linear(64→output)`. Same idea as the §3 model, just more compact.
- **`nn.CrossEntropyLoss()`** — classification loss (also the standard for language modeling).
- **`optim.AdamW(model.parameters(), lr=learning_rate)`** — the optimizer tuning all weights.
- **`for epoch in range(epochs)`** — repeat the cycle for each pass over the data.
- **`model.train()`** — put the model in **training mode** (activates training-only behaviours like dropout). The counterpart to `model.eval()` used for inference.
- **`optimizer.zero_grad()`** — clear leftover gradients from the previous step.
- **`predictions = model(X_train)`** — **forward pass**.
- **`loss = criterion(predictions, y_train)`** — measure error against the labels.
- **`loss.backward()`** — **backpropagation** computes all gradients.
- **`optimizer.step()`** — update the weights one step downhill.
- **`print(... loss.item() ...)`** — `.item()` pulls the single loss number out of the tensor so it can be printed. Watching it shrink across epochs is how you confirm the model is actually learning.

> **The order matters:** `zero_grad → forward → loss → backward → step`. Forgetting `zero_grad` makes gradients pile up; calling `step` before `backward` updates with no gradients.

---

# §8 · Efficient Training Optimization

## `gradient_accumulation.py`
This demonstrates two tricks for training **big** models on **small** hardware: **gradient accumulation** (fake a big batch by adding up several small ones) and **mixed precision** (use faster 16-bit math safely).

```python
desired_batch_size = 64
micro_batch_size = 16
accumulation_steps = desired_batch_size // micro_batch_size   # = 4
```
- **`desired_batch_size = 64`** — the batch size we *want* for stable training.
- **`micro_batch_size = 16`** — the biggest batch our GPU memory can actually hold.
- **`accumulation_steps = 64 // 16`** — so we process **4** small batches and combine them to *simulate* one batch of 64. (`//` is integer division.)

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
scaler = torch.cuda.amp.GradScaler(enabled=torch.cuda.is_available())
```
- **`torch.device("cuda" if ... else "cpu")`** — use the GPU (`cuda`) if one exists, otherwise the CPU.
- **`model.to(device)`** — move the model's weights onto that device.
- **`GradScaler(...)`** — the **mixed-precision** helper. 16-bit numbers can be too small and round to zero ("underflow"); the scaler multiplies the loss up before backprop and back down after, protecting the tiny gradients.

```python
optimizer.zero_grad()
for step in range(1, accumulation_steps + 1):
    X_micro = torch.randn(micro_batch_size, input_size).to(device)
    y_micro = torch.randint(0, output_size, (micro_batch_size,)).to(device)
    with torch.cuda.amp.autocast(enabled=torch.cuda.is_available()):
        predictions = model(X_micro)
        loss = criterion(predictions, y_micro)
        loss = loss / accumulation_steps
    scaler.scale(loss).backward()
    if step % accumulation_steps == 0:
        scaler.step(optimizer)
        scaler.update()
        optimizer.zero_grad()
```
- **`optimizer.zero_grad()` (before the loop)** — start with clean gradients; note we **don't** zero inside every step, because we *want* them to accumulate.
- **`for step in range(1, accumulation_steps + 1)`** — loop over the 4 micro-batches.
- **`X_micro = torch.randn(...).to(device)`** / **`y_micro = torch.randint(...).to(device)`** — make a small fake batch (16 examples) and move it to the GPU/CPU.
- **`with torch.cuda.amp.autocast(...)`** — run the forward pass in **mixed precision**: PyTorch automatically uses fast 16-bit math where it's safe and 32-bit where it's needed.
- **`predictions = model(X_micro)`** / **`loss = criterion(...)`** — usual forward pass and loss on the micro-batch.
- **`loss = loss / accumulation_steps`** — **critical line**: divide the loss by 4. Since we'll add up 4 batches' gradients, dividing first keeps the total equal to an *average* over 64 examples instead of a 4×-too-large *sum*.
- **`scaler.scale(loss).backward()`** — scale the loss up (mixed-precision safety) and backprop. Gradients **accumulate** into the weights' `.grad` because we're not zeroing them between micro-batches. Note: no `optimizer.step()` yet.
- **`if step % accumulation_steps == 0:`** — only true on the 4th step (when `step` is a multiple of 4) → time to actually update.
  - **`scaler.step(optimizer)`** — unscale the gradients and take the optimizer step (update weights).
  - **`scaler.update()`** — adjust the scaler's multiplier for next time.
  - **`optimizer.zero_grad()`** — now clear gradients to start the next big batch fresh.

> **Link to the markdown:** this is §8's "train big models on small GPUs" made real — accumulation gives you a large effective batch without the memory cost, and mixed precision (AMP) roughly doubles speed and halves memory.

---

# 🧠 The pattern across all eight files

```
§1 build the math/data tools  →  §2 classic ML (fit/predict)  →  §3 the neuron + training loop
   →  §4 the architectures (CNN, LSTM, Attention)  →  §5 run a real pretrained LLM
   →  §6 how text becomes tokens  →  §7 the training loop again, watched live
   →  §8 make that training fit on real hardware
```

Every script is some combination of four moves: **get data → define a model → run a fit/loop → evaluate or use it.** Once you can spot those four parts, you can read any machine-learning code.
