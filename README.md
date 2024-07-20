# Groth16 Setup

#### Project starts with circuit.circom MiMC5 circuit


## Phase 1: Ceremony - Powers of Tau

### Starting the ceremony by creating a new ceremony file 

Name of the ceremony: powersoftau,

Eliptic curve for polynomial evaluation: bn128,

Max number of constraints: 2^12

Name of the ceremony file: ceremony_0000.ptau

    snarkjs powersoftau new bn128 12 ceremony_0000.ptau



### Start adding randomness to the ceremony file. In practice this is done by 3rd part contributors (no one should have knowledge of what they have contributed and these input contributions should be discarded)

Input ceremony file: ceremony_0000.ptau

Generate new ceremony file with an entropy input contribution: ceremony_0001.ptau 

    snarkjs powersoftau contribute ceremony ceremony_0000.ptau ceremony_0001.ptau -v


Input ceremony file: ceremony_0001.ptau

Generate new ceremony file with an entropy input contribution: ceremony_0002.ptau 

    snarkjs powersoftau contribute ceremony ceremony_0001.ptau ceremony_0002.ptau -v



### Verifying a ceremony file to ensure that it is not corrupted before allowing additional randomness to be added

Input ceremony file to verify : ceremony_0002.ptau

Valid if response is: "Powers of Tau Ok!"

    snarkjs powersoftau verify ceremony_0002.ptau

---

## Phase 2: Preparing the ZKey 

### Prepare the last recieved ceremony file ready for phase 2 of Groth16 Setup

Input ceremony file: ceremony_0002.ptau

Output prepared for phase2 ceremony file: ceremony_final.ptau

    snarkjs powersoftau prepare phase2 ceremony_0002.ptau ceremony_final.ptau -v



### Verify the prepared final ceremony file 

Input ceremony file to verify : ceremony_final.ptau

Valid if response is: "Powers of Tau Ok!"
        
    snarkjs powersoftau verify ceremony_final.ptau



### Compile the circom circuit that includes the MiMC5 hashing algorithm

Compile the circuit as a rank 1 constraint system and compile it out to web assembly

    circom circuit.circom --r1cs --wasm



### Feed the MiMC5 hashing circuit to the groth16 ceremony file

Input r1cs file: circuit.r1cs

Input ceremony file: ceremony_final.ptau

Output zkey proving key file: setup_0000.zkey

    snarkjs groth16 setup circuit.r1cs ceremony_final.ptau setup_0000.zkey



### Contribute randomness to the zkey file

Input zkey file: zkey_0000.zkey

Generate new zkey file with an entropy input: setup_final.zkey

    snarkjs zkey contribute setup_0000.zkey setup_final.zkey



### Verify the zkey file to ensure it is not corrupted

Input the circuit rank 1 constraint system file: circuit.r1cs 

Input the final ceremony file: ceremony_final.ptau

Input the final zkey file: setup_final.zkey

Valid if response is: "ZKey Ok!"

    snarkjs zkey verify circuit.r1cs ceremony_final.ptau setup_final.zkey

---

## Phase 3: Generating the proof

Create the input.json file for the circuit: inputs (x, k)



### Constructing the proof 

Input inputs file for the circuit: inputs.json

Input MiMC5 circuit WASM file: circuit.wasm

Input final zkey file: setup_final.zkey

Output the proof file containing the points to compare: proof.json

Output the public file containing the circuit output (hash value from the circuit execution): public.json


    snarkjs groth16 fullprove input.json circuit_js/circuit.wasm setup_final.zkey proof.json public.json

---

## Porting the verification ability onto the blockchain

### Now we need to put a verifier on the blockchain so that we can validate proofs to be true or false 

Input final zkey: setup_final.zkey
 
Output solidity verifier file: Verifier.sol

        snarkjs zkey export solidityverifier setup_final.zkey Verifier.sol



### Deploy Verifier.sol contract on EVM blockchain



### Generate call data for verification by the Verifier smart contract

Input the public output file containing the hashed output data from the MiMC5 circuit: public.json

Output the calldata to a file: soliditycalldata

    snarkjs zkey export soliditycalldata public.json proof.json > soliditycalldata

We can now input this call data into the Verifier smart contract and it should return true.
If we were to modify the call data, the Verifier would return false.

