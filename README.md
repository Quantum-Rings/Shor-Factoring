# Shor's Algorithm

This folder hosts our implementation(s) of **Shor’s Algorithm** for integer factoring on a quantum computer. Shor’s algorithm famously demonstrates an exponential speedup over the best-known classical factoring algorithms under ideal conditions, making it a cornerstone example of quantum computing’s disruptive potential.

## 1. Overview of Shor’s Algorithm

- **Goal**: Factor a semiprime integer N into its prime factors using quantum phase estimation and modular arithmetic.
  
- **Key Components**:
  

  1. **Quantum Phase Estimation (QPE)**: Uses the [Quantum Fourier Transform](https://en.wikipedia.org/wiki/Quantum_Fourier_transform) (QFT) to estimate the order of \\( a \\) modulo \\( N \\).

  2. **Modular Exponentiation**: A sequence of controlled multiplications and modular reductions, typically built from lower-level circuits like _adders_ and _multipliers_.

  3. **Classical Post-Processing**: Once you measure a candidate for the period (the order), you use classical gcd checks to extract factors.

In practice, we break these tasks into smaller _building blocks_ that are reusable in many quantum algorithms.

## 2. Fully Contained code or Subsystems Modules

Code can be self contained examples or call in any of the available helper code.

1. Include code header describing the impmenentation.  
  
2. Also include the tested module versions.
  

Example:

---

**Concept**

Provide information about the concept of the code and how to use it. Is the goal to use fewer qubits, shorter circuit depth, faster execution, more accurate period finding, etc. When you run the code, how do you set N? Are the "a" vlaues randomly selected, in a list, input as a parameter?

**Environment & Dependencies**

- **Python**: 3.11
  

- **QuantumRingsLib**: 0.10.0
- **Operating System**: Windows 11 (or equivalent Linux/macOS/Windows)

---

### Example Flow

1. **Register Setup**: Allocate qubits for QPE, and a “work” or “value” register to hold the modular states.
  
2. **Superposition**: Place the “exponent” register in superposition (`Hadamard` gates).
  
3. **Modular Exponentiation**: For each qubit in the exponent register, conditionally multiply the “work” register by \\( a^{2^i} \mod N \\).  
  
4. **Inverse QFT**: Apply the inverse QFT on the exponent register.
  
5. **Measurement**: Measure the exponent register to obtain the periodic information.
  
6. **Classical Post-Processing**: Compute gcd and check for nontrivial factors.
  

Each step calls into building blocks—e.g., `qft_circuit()`, `modular_multiply()`, or specific adder gates.

## 3. Subsystems

### Adders

#### Carry-Lookahead Adder

Toffoli-Based “Carry-Lookahead” Adder

- An approach that tries to parallelize part of the carry operation.
- More advanced, sometimes less widely used in teaching materials.

#### CDKM Adder#

- Proposed by Takahashi et al. (sometimes attributed to Takahashi-Kunihiro or Draper-Kutin).
- Known to reduce the circuit depth compared to a naive ripple-carry approach.

#### Cuccaro Adder

Ripple-Carry (Cuccaro) Adder

- Sometimes called the “quantum ripple-carry adder” or the Cuccaro adder.
- It’s quite straightforward but not the most gate-efficient.
- Often used as a teaching example.

Reference

- [A new quantum ripple-carry addition circuit](https://arxiv.org/abs/quant-ph/0410184), *Steven A. Cuccaro, Thomas G. Draper, Samuel A. Kutin, David Petrie Moulton* (2004)

#### Draper Adder

- A QFT-based adder, as introduced by Thomas Draper.
- It uses the Quantum Fourier Transform to implement addition with fewer carry operations but typically involves more non-local gates.

Reference

- [Addition on a Quantum Computer](https://arxiv.org/abs/quant-ph/0008033), *Thomas G. Draper* (2000)

#### QCLA Adder

Quantum Carry-Lookahead (QCLA) Adder

Reference

- [A logarithmic-depth quantum carry-lookahead adder](https://arxiv.org/abs/quant-ph/0406142), *Thomas G. Draper, Samuel A. Kutin, Eric M. Rains, Krysta M. Svore* (2004)

#### VBE Adder

Vedral-Barenco-Ekert (VBE) Adder

- Typically an improvement over naive ripple-carry.

Reference:

- [Quantum Networks for Elementary Arithmetic Operations](https://arxiv.org/abs/quant-ph/9511018), *V. Vedral, A. Barenco, A. Ekert* (1995)

### Modular Exponentiation

#### Carry-Lookahead Multiplier

Concept

- In classical computing, we use carry-lookahead techniques to reduce carry-propagation time.
- The quantum analog aims to parallelize or reduce the overhead of successive carry bits.
- Some references adapt “Wallace tree” or other parallel classical multipliers to a quantum circuit.

Pros

- Potentially lower circuit depth than naive ripple-carry-based multipliers.
- Good for large integer multiplication where speed is key.

Cons

- Usually complex to design in a fully reversible manner.
- May require more ancillas.

#### CDKM Multiplier

CDKM or Vedral-Barenco-Ekert (VBE) Multiplier

Concept

- These multipliers are often part of broader quantum arithmetic frameworks proposed in the same papers that ntroduced improved adders.
- “CDKM” is from Draper, Kutin, Rains, Svore (sometimes referencing Takahashi-Kunihiro improvements).
- VBE is from Vedral, Barenco, and Ekert (1996), originally describing an adder, but has extensions to ultiplication.

Pros

- Typically optimized for fewer gates or lower depth than naive approaches.
- Well-documented in quantum computing literature.

Cons

- Implementation complexity is higher.
- Harder to find straightforward references or code examples compared to simpler “repeated addition.”

#### Draper Multiplier

QFT-Based (Draper) Multiplier

Concept

- Utilizes the Quantum Fourier Transform to perform multiplication in the frequency domain.
- Similar spirit to the Draper adder, but extended to handle multiplication.

Pros

- Potentially fewer carry operations in the time domain.
- Shows off QFT-based arithmetic as a “clean” conceptual approach.

Cons

- Requires many controlled-phase gates.
- May be sensitive to approximations or truncated phases to be practical.

#### Karatsuba Multiplier

Karatsuba or Toom-Cook Multipliers (Asymptotically Faster Methods)

Concept

- In classical arithmetic for large numbers, Karatsuba multiplication reduces the complexity below 𝑂(𝑛^2).
- Toom-Cook (and further generalizations like Schönhage–Strassen in the classical domain) continue to reduce symptotic complexity.
- Quantum analogs exist that try to replicate these divide-and-conquer strategies.

Pros

- Lower asymptotic complexity for very large 𝑛.
- Potential for significantly fewer operations if implemented well.

Cons

- Implementation can be complicated.
- For small to medium 𝑛, overhead might outweigh the theoretical benefits.

#### Shift & Add Multiplier

Repeated-Addition (Shift-and-Add) Multiplier

Concept

- The most straightforward approach: multiply 𝑎 by 𝑏 by repeatedly adding 𝑎 to an accumulator if the corresponding bit of 𝑏 is 1, while shifting.
- Often taught in classical computing courses as the simplest CPU-level multiplier, adapted to quantum by making ach addition reversible.

Pros

- Easy to implement and reason about.
- Good for teaching or demonstration.

Cons

- Not the most gate-efficient.
- Large circuit depth for bigger operands.

#### Ragavan Modulation

Space‑Efficient & Noise‑Robust Quantum Factoring

*(Ragavan & Vaikuntanathan)*

An improved variant of Regev’s quantum factoring algorithm that achieves the “best of both worlds” by

1. **Reducing qubit overhead** to (O(n\log n)) while keeping gate count (\tilde O(n^{3/2})), via **reversible Fibonacci‑chain exponentiation**, and
2. **Tolerating noisy runs** by **filtering out** corrupted period samples through lattice‑reduction diagnostics.

References

- [Space‑Efficient and Noise‑Robust Quantum Factoring](https://arxiv.org/abs/2310.00899), *Seyoon Ragavan, Vinod Vaikuntanathan* (2023)
- [An Efficient Quantum Factoring Algorithm](https://arxiv.org/abs/2308.06572), *Oded Regev* (2023)
- [Targeted Fibonacci Exponentiation](https://arxiv.org/abs/1711.02491), *Burton S. Kaliski Jr.* (2017)
- [A new quantum ripple‑carry addition circuit](https://arxiv.org/abs/quant-ph/0410184), *Steven A. Cuccaro, Thomas G. Draper, Samuel A. Kutin, David Petrie Moulton* (2004)

### Phase Analysis

#### QFT (iQFT)

#### Approximations

- **Approximate QFT**: Reduces the number of small-phase gates.
  
  - Also called: Truncated QFT, Phase-Truncated QFT
    
    - Description: Reduces the number of phase gates by truncating small rotations, speeding up the QFT at the cost of slight approximation error. Commonly used in Shor’s algorithm to reduce circuit depth (especially for large registers). 
- **Semiclassical QFT (or iQFT)**
  
  - Also called: Semi-classical Phase Estimation, Measure-and-Feedforward QFT
    
    - Description: As you measure each qubit (starting from the least significant), you apply corrections to subsequent qubits, thereby eliminating many controlled gates. This technique can be a big gate-count saver for phase estimation (and thus for Shor).
- **Windowed Exponentiation**: Splits exponent bits into chunks to reduce gate depth.  
  
- **Non-reversible Adders**: Potentially lowers gate counts at the cost of more ancilla management.
  

These improvements swap in different subsystem modules with minimal changes to the orchestration logic.

---

## 4. Usage / How to Run

**Example (Naive Implementation)**

```
cd algorithms/shor

python src/shor_naive.py --N 15 --a 2
```

This might print logs about the circuit construction, run a simulator, or query a quantum backend, then output found factors. For more usage details:

```
python src/shor_naive.py --help
```

Example (Optimized Implementation)

```
cd algorithms/shor

python src/shor_optimized.py --N 21 --a 2
```

Adjust flags if needed to select approximate QFT, window size, etc.

## 5. Notebooks and Demos

shor\_demo.ipynb: Walks through Shor’s algorithm step by step, visualizing the circuit and measuring an example like ( N=15 ).

advanced\_shor\_comparison.ipynb: Compares naive vs. approximate QFT in terms of circuit depth, gate count, or runtime performance.

Open these notebooks in Jupyter or JupyterLab to interactively run them and see plots or circuit diagrams.

## 6. Extending or Contributing

We welcome fixes, new examples, and improvements:

1. **Fork** the repo and create a descriptive feature branch.
2. **Write** your code or documentation, following the Example Guidelines above.
3. **Test** locally to ensure it runs on Quantum Rings.
4. **Commit** with a clear message and push your branch.
5. **Open a Pull Request** against `main`—we’ll review, iterate, and merge.

Try new adders or multipliers then import them in your Shor version to see how they impact performance.

Notebook tutorials: If you develop a step-by-step explanation or advanced Shor variations, add a new notebook in the notebooks/ folder.

We welcome pull requests and feedback

## 7. References

A few key references on Shor’s algorithm and relevant optimizations:

- Peter W. Shor, “Algorithms for quantum computation: discrete logarithms and factoring,” Proceedings 35th Annual Symposium on Foundations of Computer Science, 1994.
  
- Thomas G. Draper, “Addition on a Quantum Computer,” arXiv:quant-ph/0008033, 2000.
  
- Beauregard, S. (2003). “Circuit for Shor’s Algorithm Using 2n+3 Qubits.” arXiv:quant-ph/0205095.
  
- Nielsen & Chuang, Quantum Computation and Quantum Information, 10th Anniversary Edition.
  

For a deeper dive into the building blocks, see subsystems/README.md and the docs in each subfolder (adders, multipliers, qft, etc.).

Enjoy exploring Shor’s algorithm! If you have questions or want to propose an improvement, feel free to open an issue or start a discussion in this repository.

---

### Final Tips

- Adjust _paths_, _filenames_, and _scripts_ to match your actual code.
  
- Add or remove **sections** to reflect your project’s real content—e.g., if you only have `shor.py` with a single approach, or if you have multiple advanced versions, rename them accordingly.
  
- Include **build instructions** or **installation steps** (like `conda` environment usage) if relevant to running Shor’s code in your environment.
  
  <!--  -->
  

## License

All examples and documentation are released under the **MIT License**.
