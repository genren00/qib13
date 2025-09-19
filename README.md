# qib13
A mnemonic-like tool for qubic (can be adopted to other coins) I recommend running the html offline for min maxing security


🔴🔴🔴🔴please dont use aaaaaa's as a generator code , its insecure 🔴🔴🔴

updated rng to HMAC Crypto rng , see (better rng commit for clarification)




# qib13 Documentation

## Overview

qib13 is a reproducible multi-deviation seed recovery system that generates multiple encoding systems from a master seed. The system transforms a 13-character generator code into a 55-character seed using one of several encoding systems. The same generator code can be recovered from the seed using the same master seed, even with multiple deviation systems available for recovery.

The system is designed for high reliability and memorability, allowing users to work with simple generator codes while maintaining cryptographic security through a master seed.

## System Architecture

The qib13 system consists of the following core components:

1. **Master Seed**: A string of any length that serves as the root of trust and enables deterministic generation of all encoding systems.
2. **Generator Code**: A 13-character string (a-z) that is encoded into a seed.
3. **Encoding Systems**: Multiple systems (one original and multiple deviations) generated deterministically from the master seed.
4. **Seed**: A 55-character string produced by encoding the generator code using one of the encoding systems.

The system operates in two primary modes:
- **Encoding**: Transform a generator code into a seed using a specific encoding system.
- **Decoding**: Recover the original generator code from a seed using one or more encoding systems.

## Key Components

### Master Seed

The master seed is a string of any length that serves as the foundation for generating all encoding systems. It must be kept secure as it is the primary source of entropy for the system. While the system has a default master seed, users can and should use their own custom master seeds for their specific applications.

Example: `my-custom-secret-key-2023` or `this-is-a-very-long-and-secure-master-seed-that-i-created`

### Generator Code

A 13-character string containing only lowercase letters (a-z). This code is transformed into a seed by the encoding systems. The generator code can be simple and memorable, such as repeated characters or meaningful patterns.

Examples:
- `zzzzzzzzzzzzz`(not recommended)
- `abcdefghijklm`
- `mynamepattern`

### Encoding Systems

Multiple encoding systems are generated deterministically from the master seed:
- **Original System**: The first encoding system, created using the master seed with the suffix "-original-1".
- **Deviation Systems**: Additional encoding systems created using the master seed with suffixes "-deviation-N" where N ranges from 2 to the number of systems + 1.

Each encoding system consists of:
- **Matrices**: For original systems, a single matrix P; for deviation systems, two matrices P1 and P2.
- **Permutation**: A random permutation of 55 elements.
- **Inverse Permutation**: The inverse of the permutation, used for decoding.

### Seed

A 55-character string containing only lowercase letters (a-z) produced by encoding a generator code. The seed contains the original generator code along with parity information that enables error detection during recovery.

## Security Analysis

### Entropy Calculation

- **Master Seed Entropy**: L × log₂(26) bits, where L is the length of the master seed
- **Generator Code Entropy**: 13 characters × log₂(26) ≈ 61.1 bits
- **Deviation Index Entropy**: log₂(N) bits, where N is the number of systems
- **Total System Entropy**: (L × log₂(26)) + 61.1 + log₂(N) bits

For a typical 80-character master seed with 10 systems:
- Master seed entropy: 80 × log₂(26) ≈ 376 bits
- Generator code entropy: 61.1 bits
- Deviation index entropy: log₂(10) ≈ 3.3 bits
- Total entropy: 440.4 bits

### Security Model

The security of qib13 depends on the secrecy of the master seed:

1. **With Master Seed Secrecy**: The system provides substantial entropy, making it computationally infeasible to brute-force.
2. **With Master Seed Compromise**: Security reduces to the generator code entropy plus deviation index entropy (61.1 + log₂(N) bits), which may be vulnerable to brute-force attacks.

The deviation systems add entropy through the uncertainty of which system was used to encode the seed. The same generator code can produce N different seeds (one per system), making it harder to identify patterns or correlations.

### Threat Model

- **External Attackers**: Cannot recover generator codes without the master seed.
- **Internal Attackers**: With access to the master seed, can attempt to brute-force generator codes across all deviation systems.
- **Data Corruption**: Multiple deviation systems provide resilience against corrupted seeds.

## Usage Examples

### Example 1: Basic Encoding and Decoding with Custom Master Seed

1. **Generate Systems**:
   - Master seed: `my-custom-secret-key-2023`
   - Number of systems: 10

2. **Encode Generator Code**:
   - Generator code: `zzzzzzzzzzzzz`
   - Mode: Original (System 1)
   - Generated seed: `qibkzrjxqyvzjwqfjxkqzrjxqyvzjwqfjxkqzrjxqyvzjwqfjxkqzrjxqyvzjwqf`

3. **Decode Seed**:
   - Seed: `qibkzrjxqyvzjwqfjxkqzrjxqyvzjwqfjxkqzrjxqyvzjwqfjxkqzrjxqyvzjwqf`
   - Mode: Sequential Recovery (try all systems)
   - Recovered generator code: `zzzzzzzzzzzzz`
   - Successful system: System 1 (Original)

### Example 2: Using a Specific Deviation with a Different Master Seed

1. **Generate Systems**:
   - Master seed: `this-is-a-very-long-and-secure-master-seed-that-i-created`
   - Number of systems: 20

2. **Encode Generator Code**:
   - Generator code: `abcdefghijklm`
   - Mode: Specific System (System 15)
   - Generated seed: `mnbvcxzlkjhgfdsapoiuytrewqmnbvcxzlkjhgfdsapoiuytrewqmnbvcxzlkjh`

3. **Decode Seed**:
   - Seed: `mnbvcxzlkjhgfdsapoiuytrewqmnbvcxzlkjhgfdsapoiuytrewqmnbvcxzlkjh`
   - Mode: Specific System (System 15)
   - Recovered generator code: `abcdefghijklm`

### Example 3: Batch Testing with a Short Master Seed

1. **Generate Systems**:
   - Master seed: `short-key`
   - Number of systems: 5

2. **Run Batch Test**:
   - Number of test cases: 5
   - Results:
     - Successful recoveries: 5
     - Total attempts: 15
     - Success rate: 100%
     - Master seed: `short-key`

## Implementation Details

### Seeded Random Number Generation

The system uses a HMAC-SHA random number generator to ensure deterministic-CRYPTOGRAPHICALLY SECURE generation of encoding systems:


### Matrix Generation

Matrices are generated using the seeded random number generator:

```javascript
function generateRandomMatrix(rows, cols, rng) {
    const matrix = [];
    for (let i = 0; i < rows; i++) {
        const row = [];
        for (let j = 0; j < cols; j++) {
            row.push(rng.nextInt(0, 26));
        }
        matrix.push(row);
    }
    return matrix;
}
```

### Permutation Generation

Permutations are generated using the Fisher-Yates shuffle algorithm:

```javascript
function generateRandomPermutation(n, rng) {
    const perm = Array.from({length: n}, (_, i) => i);
    
    for (let i = n - 1; i > 0; i--) {
        const j = rng.nextInt(0, i + 1);
        [perm[i], perm[j]] = [perm[j], perm[i]];
    }
    
    return perm;
}
```

### Encoding Process

The encoding process transforms a generator code into a seed:

1. Convert the generator code to numbers.
2. Apply matrix multiplication to generate parity information.
3. Combine the original message with negated parity information.
4. Apply a permutation to the combined data.
5. Convert the result back to letters.

### Decoding Process

The decoding process recovers the generator code from a seed:

1. Convert the seed to numbers.
2. Apply the inverse permutation.
3. Separate the message and parity information.
4. Verify the parity information using matrix multiplication.
5. Convert the message back to letters.

## Advantages

### High Reliability

The multi-deviation approach provides multiple recovery paths, dramatically improving the chances of successful recovery. With N systems, the probability of all systems failing is p^N, where p is the failure probability of a single system.

### Memorability

Users can work with simple, memorable generator codes (e.g., "zzzzzzzzzzzzz") while maintaining security through the master seed. This significantly reduces the cognitive burden compared to remembering complex random sequences.

### Deterministic Generation

The same master seed will always generate the same set of encoding systems, ensuring reproducibility across different sessions and devices.

### Configurable Security

Users can adjust the number of deviation systems based on their reliability requirements. More systems provide higher reliability and slightly more entropy at the cost of increased storage and computation.

### Flexible Master Seed

The system accepts master seeds of any length, allowing users to balance security and convenience. Longer master seeds provide more entropy, while shorter ones are easier to manage.

## Limitations

### Master Seed Dependency

The security of the system depends entirely on the secrecy of the master seed. If the master seed is compromised, the generator codes become vulnerable to brute-force attacks.

### Storage Requirements

The master seed must be stored securely, especially for longer master seeds that cannot be easily memorized. This requires a secure storage solution.

### Computational Overhead

Recovery using multiple systems requires additional computation, as each system must be tested sequentially until a successful recovery is achieved.

### Generator Code Entropy

The generator code has limited entropy (61.1 bits), making it vulnerable to brute-force attacks if the master seed is compromised. Users should avoid using predictable generator codes in high-security scenarios.

## Conclusion

qib13 provides a robust solution for encoding and recovering generator codes with high reliability and memorability. By separating the concerns of security (handled by the master seed) and usability (handled by simple generator codes), the system offers a practical approach to seed management that balances security with user experience. The multi-deviation architecture ensures reliable recovery even in error-prone environments, making it suitable for applications where data integrity is critical. The flexible master seed implementation allows users to customize the system to their specific security requirements.
