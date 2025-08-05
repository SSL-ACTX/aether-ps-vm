<br/>
<p align="center">
  <img src="https://i.ibb.co/YT0BhPr0/img.jpg" alt="Aether-VM Logo" width="150">
  <h1 align="center">Aether-VM Protector</h1>
  <p align="center">
    A "simple" JavaScript obfuscation tool that transforms critical code into a self-mutating (pseudo), polymorphic virtual machine.
    <br />
    <br />
    <a href="#-key-features"><strong>Explore the Features »</strong></a>
    <br />
    <br />
    <a href="#-getting-started">Installation</a>
    ·
    <a href="#-usage">Usage Guide</a>
    ·
    <a href="#-how-it-works-a-technical-deep-dive">Technical Deep Dive</a>
  </p>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/badge/License-Proprietary-red?style=for-the-badge">
  <img alt="Version" src="https://img.shields.io/badge/Version-1.2.0-blue?style=for-the-badge">
  <img alt="Node.js" src="https://img.shields.io/badge/Node.js-%3E%3D16.x-green?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/Platform-CLI-lightgrey?style=for-the-badge">
</p>

---

## ⚠️ Extremely Important Disclaimer

**This software is provided for educational and research purposes only.** It is a proof-of-concept demonstrating advanced code obfuscation techniques. While it aims to make reverse-engineering more difficult, **it does not guarantee absolute security.**

A determined and skilled adversary can, with sufficient time and effort, reverse-engineer any client-side protection. This tool should not be relied upon as the sole means of protecting sensitive data, intellectual property, or critical business logic. True security is achieved through robust server-side architecture and validation. The author assumes no liability for any misuse or for any security breaches that may occur in software protected by this tool. **Use at your own risk.**

> 🚫 Aether-VM is NOT to be used for malware, cheats, botnets, or any other shady BS. This project exists to protect *legitimate* intellectual property — not hide exploits.

---

## About The Project

**Aether-VM** is a standalone command-line tool designed to provide an advanced layer of protection for JavaScript code. Instead of simple minification or string encoding, Aether-VM converts targeted code blocks—or even entire scripts—into a secure, self-mutating virtual machine.

This process involves:
1.  **Compilation:** Your JavaScript code is compiled into a custom, encrypted bytecode.
2.  **Virtualization:** A unique, polymorphic VM interpreter is generated to execute this bytecode.
3.  **Embedding:** The VM and the encrypted bytecode are embedded back into your script.

When the protected script is executed, the VM interpreter decrypts and runs the bytecode, effectively hiding the original logic from static analysis and casual inspection.

## ✨ Key Features

*   **Dual Protection Modes:**
    *   **Block Mode:** Protect specific, critical sections of your code using `/* @vm-protect-start */` and `/* @vm-protect-end */` tags.
    *   **Extreme Mode:** Virtualize the entire script file into a single, monolithic VM for maximum obfuscation.
*   **Polymorphic VM Engine:** No two generated VMs are the same. The interpreter's internal logic, variable names, and control flow are randomized on every run.
*   **Self-Mutating Opcodes:** The mapping of bytecode to its function changes dynamically at runtime, frustrating attempts to build a static disassembler.
*   **Layered Obfuscation:**
    *   **AES-128-GCM Encryption:** The core bytecode is encrypted, and its integrity is verified with an authentication tag.
    *   **Logic Atomization:** Critical strings (like 'crypto', 'decrypt', etc.) are removed and instead constructed on-the-fly from XOR-encrypted byte arrays.
    *   **Anti-Tampering Traps:** Fake, decoy instructions are injected into the bytecode to mislead and trap reverse-engineers.
*   **Built-in Anti-Debugging:**
    *   Detects debugger presence through timing checks (`performance.now()`).
    *   Detects DevTools being opened via property access monitoring.
    *   Integrity checks to ensure the VM's code has not been modified (e.g., by "prettifying" it in a debugger).
*   **Layering Support:** In Extreme Mode, you can apply multiple layers of virtualization, wrapping a VM inside another VM for exponentially increased complexity.
*   **Terser Integration:** Seamlessly minify or prettify the final output for production deployment or analysis.

## ⚙️ Getting Started

### Prerequisites

*   Node.js (v16.x or higher)
*   npm or yarn

### Installation

1.  Clone the repository:
    ```sh
    git clone https://github.com/SSL-ACTX/aether-ps-vm.git
    cd aether-ps-vm
    ```

2.  Install the required dependencies:
    ```sh
    npm install
    ```
    or
    ```sh
    yarn install
    ```
3. Make the CLI tool executable (on Linux/macOS):
   ```sh
   chmod +x vm-cli.cjs
   ```

## 🚀 Usage

The tool is operated via the `vm-cli.cjs` script.

### Basic Command Structure

```sh
./vm-cli.cjs <input> <output> [options]
```

| Argument | Description                                 |
| :------- | :------------------------------------------ |
| `input`  | Path to the input JavaScript file.          |
| `output` | Path where the protected output will be saved. |

### Protection Modes

#### 1. Block Mode (Default) - Still WIP [UNSTABLE]

This is the standard mode. You wrap the code you want to protect in special comments. The tool will find these blocks, generate a single VM interpreter, and replace each block with a payload that runs on that VM.

**Example Input (`my-app.js`):**

```javascript
function showPublicMessage() {
  console.log("This is a public function, everyone can see this.");
}

/* @vm-protect-start */
// This logic is highly sensitive and will be virtualized.
const apiKey = "ASC-123-XYZ";
const secretValue = Math.random() * 1000;

if (secretValue > 500) {
  console.log("Performing secret action with:", apiKey);
}
/* @vm-protect-end */

showPublicMessage();

/* @vm-protect-start */
function anotherSecretCalculation(a, b) {
  // This function will also be moved into the VM.
  return (a * b) ^ 0xDEADBEEF;
}
console.log("Secret result:", anotherSecretCalculation(10, 20));
/* @vm-protect-end */
```

**Command:**

```sh
./vm-cli.cjs my-app.js protected/my-app.protected.js
```

The output file will contain one VM interpreter and two encrypted payloads replacing the original blocks.

#### 2. Extreme Mode

This mode ignores the `@vm-protect` blocks and virtualizes the entire input file. This is useful for protecting entire libraries or standalone scripts.

**Command:**

```sh
./vm-cli.cjs my-app.js protected/my-app.extreme.js --extreme
```

### Options

| Flag                        | Alias | Type      | Default | Description                                                                                    |
| :-------------------------- | :---- | :-------- | :------ | :--------------------------------------------------------------------------------------------- |
| `--extreme`                 |       | `boolean` | `false` | Virtualize the entire script, ignoring `/* @vm-protect */` blocks.                              |
| `--layers`                  | `-l`  | `number`  | `1`     | Number of VM layers to apply. **Only effective in `--extreme` mode.**                          |
| `--minify`                  | `-m`  | `boolean` | `true`  | Minify the final output with Terser. This is the default behavior.                               |
| `--prettify`                | `-p`  | `boolean` | `false` | Format the final output for readability. Disables minification.                                  |
| `--hook`                    |       | `boolean` | `false` | **For Analysis Only.** Enable VM console logs for debugging mutations.                           |
| `--trap-density`            |       | `number`  | `0.4`   | Probability (0.0 to 1.0) of injecting a fake instruction (anti-tamper trap).                     |
| `--fake-opcodes`            |       | `number`  | `20`    | Number of different fake opcodes to generate for the VM.                                       |
| `--mutation`                |       | `number`  | `0.35`  | Probability (0.0 to 1.0) of injecting an opcode mutation instruction after a real instruction. |

### Advanced Example: Layered Extreme Protection

To create a highly complex, multi-layered VM, use `--extreme` with `--layers`.

```sh
# Apply 3 layers of virtualization to the entire script
./vm-cli.cjs my-library.js dist/my-library.protected.js --extreme --layers 3
```
This command will:
1. Wrap `my-library.js` in a VM (Layer 1).
2. Wrap the Layer 1 VM in another VM (Layer 2).
3. Wrap the Layer 2 VM in a final VM (Layer 3).
The resulting file will be extremely difficult to analyze.

## 🔬 How It Works: A Technical Deep Dive

The strength of Aether-VM comes from its multi-faceted approach to obfuscation.

1.  **Bytecode Compilation:** The target JavaScript is not executed directly. It is first compiled into a custom bytecode format. This bytecode is a series of instructions for a stack-based virtual machine (e.g., `PUSH_DATA`, `XOR`, `ADD`, `INDIRECT_EXEC`).
2.  **String Obfuscation (`Logic Atomization`):** The VM interpreter is designed to have **zero hardcoded strings** related to its cryptographic or execution functions. They are pushed to the stack and decrypted on-the-fly only when needed, preventing simple `grep` or string analysis from revealing the VM's purpose.
3.  **Encrypted Payload:** The compiled bytecode is then encrypted using **AES-128-GCM**. This ensures that the bytecode is both confidential (unreadable without the key) and authentic (cannot be tampered with, thanks to the GCM auth tag). The decryption key itself is pushed to the VM's stack and is never stored as a raw variable.
4.  **Polymorphic Interpreter Generation:** The JavaScript code that acts as the VM interpreter is generated dynamically.
    *   **Randomized Identifiers:** All internal variables (`stack`, `programCounter`, etc.) are given random, mangled names.
    *   **Shuffled Logic:** The `switch` statement that handles opcodes has its `case` blocks shuffled for every build, disrupting any assumed logical order.
    *   **Fake Opcodes:** A significant number of fake, decoy opcodes are generated. These opcodes perform harmless but plausible-looking operations and are interspersed with real opcodes to serve as anti-tampering traps.
5.  **Runtime Self-Mutation:** This is a key defense mechanism. During execution, the VM can encounter a special `MUTATE` instruction. This instruction will dynamically re-map the bytecode values to different actions. For example, before mutation, `0x1A` might mean `ADD`, but after mutation, `0x1A` might mean `DUP` and `0x5C` now means `ADD`. This makes it impossible to build a single static mapping of opcodes to their functions.

The final output is a self-contained script that first defines this complex, booby-trapped interpreter and then executes it with the encrypted bytecode, creating a formidable barrier against reverse-engineering.

## 📜 License

This project is not open source. All rights are reserved by the author.

**You are NOT permitted to:**
*   Distribute, sublicense, or sell this software.
*   Modify the source code for any purpose other than personal learning.
*   Use this software for any commercial or production purposes.
*   Remove or alter any copyright or license notices.

For any other use, you must obtain explicit written permission from the author.

---
*Project by Seuriin*
