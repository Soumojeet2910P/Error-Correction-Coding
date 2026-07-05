## Implementing a Quasi-Cyclic LDPC (QC-LDPC) encoder, a hard-decision Bit-Flipping decoder, and a Monte Carlo BER-vs-Eb/N0 simulation, built up step by step from base-matrix construction through BPSK transmission over an AWGN channel.

The code is written to be configurable —  the codeword length N, message length K, and lifting factor z can be changed to experiment with different code sizes, rather than being hardcoded to one example.

### Repository Contents

#### ldpc_encoder.py

The encoder implements the full construction and encoding pipeline:

Base matrix (B) — auto-sized from N, K, z (generate_base_matrix), with a random search for a set of circulant shifts that produces a full-rank parity-check matrix.
Lifting (build_parity_check_matrix) — expands B into the full H matrix by replacing each entry with a z x z circulant permutation block (circulant_permutation_matrix).
Systematic form (to_systematic_form) — Gaussian elimination over GF(2) to reduce H into [P | I], with row/column pivoting.
Generator matrix (derive_generator_matrix) — derives G = [I | Pᵀ].
Encoding (encode) — v = u · G (mod 2).
BPSK modulation (bpsk_modulate) — maps bits to +1/-1 symbols.
AWGN channel (awgn_noise_std, awgn_channel) — adds Gaussian noise at a specified Eb/N0 (dB), correctly accounting for the code rate.

Also includes verify_all_codewords, a sanity check that encodes every
possible K-bit message and confirms H · vᵀ = 0 for all of them.

#### bf_decoder.py

The decoder implements the iterative hard-decision Bit-Flipping (BF) algorithm:

Hard-decision demodulation (hard_decision_demodulate) — converts the continuous received samples y back into a binary vector z.
Syndrome computation (compute_syndrome) — s = z · Hᵀ (mod 2).
Unsatisfied-check counting (count_unsatisfied_checks) — computes f_n, the number of failed parity checks each bit participates in.
Bit selection and flipping (bit_flipping_decode) — supports two rules:

rule="max" — flip the bit(s) with the largest f_n (with a tie_break option, see Known Limitations below)
rule="threshold" — flip every bit with f_n greater than a fixed threshold T

Message extraction (extract_message_bits) — since encoding is
systematic, the first K bits of the decoded codeword are the message.


#### ber_simulation.py

A Monte Carlo simulation that sweeps over a range of Eb/N0 values, running the full encode → modulate → AWGN → decode chain many times per point, and plots the resulting Bit Error Rate (BER) vs Eb/N0, alongside the theoretical uncoded-BPSK curve for comparison. Uses an adaptive stopping rule (stop once a target number of bit errors is observed, up to a max trial cap) so low-noise points don't require an impractical number of trials.
