# Groth16 Setup

Project starts with circuit.circom mimc5 circuit


## Phase 1: Ceremony - Powers of Tau

Starting the ceremony by creating a new ceremony file 

--> 

Name of the ceremony: powersoftau,
Eliptic curve for polynomial evaluation: bn128,
Max number of constraints: 2^12
Name of the ceremony file: ceremony_0000.ptau

    snarkjs powersoftau new bn128 12 ceremony_0000.ptau

---

Start adding randomness to the ceremony file. In practice this is done by 3rd part contributors (no one should have knowledge of what they have contributed and these input contributions should be discarded)

-->

Input ceremony file: ceremony_0000.ptau
Generate new ceremony file with an entropy input contribution: ceremony_0001.ptau 

    snarkjs powersoftau contribute ceremony ceremony_0000.ptau ceremony_0001.ptau -v


Input ceremony file: ceremony_0001.ptau
Generate new ceremony file with an entropy input contribution: ceremony_0002.ptau 

    snarkjs powersoftau contribute ceremony ceremony_0001.ptau ceremony_0002.ptau -v

---

Verifying a ceremony file to ensure that it is not corrupted before allowing additional randomness to be added

---> 

Input ceremony file to verify : ceremony_0002.ptau
Valid if response is: "Powers of Tau Ok!"

    snarkjs powersoftau verify ceremony_0002.ptau

---

## Phase 2

Prepare the last recieved ceremony file ready for phase 2 of Groth16 Setup

-->

Input ceremony file: ceremony_0002.ptau
Output prepared for phase2 ceremony file: ceremony_final.ptau

    snarkjs powersoftau prepare phase2 ceremony_0002.ptau ceremony_final.ptau -v

---

Verify the prepared final ceremony file

---> 

Input ceremony file to verify : ceremony_final.ptau
Valid if response is: "Powers of Tau Ok!"
        
    snarkjs powersoftau verify ceremony_final.ptau

---

Compile the circom circuit that includes the MiMC5 hashing algorithm

-->

Compile the circuit as a rank 1 constraint system and compile it out to web assembly

    circom circuit.circom --r1cs --wasm

---

Feed the MiMC5 hashing circuit to the groth16 ceremony file

-->

input r1cs file: circuit.r1cs
input ceremony file: ceremony_final.ptau
output zkey proving key file: setup_0000.zkey

    snarkjs groth16 setup circuit.r1cs ceremony_final.ptau setup_0000.zkey

---

Contribute randomness to the zkey file

-->

    snarkjs zkey 




