## Code journal for [PROC INSTR NAT](https://github.com/solana-developers/program-examples/blob/main/basics/processing-instructions/native/program/src/lib.rs)  <br> <br>

### What are the concepts (borrowing, ownership, vectors etc)?
The concepts utilized in the code are structs, derive attribute to extend borsh, if statement.  <br> <br>

### What is the contract doing? What is the mechanism? 
Contract is deseralizing our instruction data   <br> <br>

------
<br>

```rust
use borsh::{ BorshDeserialize, BorshSerialize };
use solana_program::{
    account_info::AccountInfo,
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    pubkey::Pubkey,
};

entrypoint!(process_instruction);

fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8]
) -> ProgramResult {
    // Attempt to deserialize the BPF format to our struct using Borsh
    let instruction_data_object = InstructionData::try_from_slice(&instruction_data)?;

    // Print msg with name from InstructionData
    msg!("Welcome to the park, {}!", instruction_data_object.name);
    // Check if height parsed from struct is over 5
    if instruction_data_object.height > 5 {
        msg!("You are tall enough to ride this ride. Congratulations.");
    } else {
        msg!("You are NOT tall enough to ride this ride. Sorry mate.");
    }

    Ok(())
}

// The compiler is capable of providing basic implementations for some traits via the #[derive] attribute.
#[derive(BorshSerialize, BorshDeserialize, Debug)]
pub struct InstructionData {
    name: String,
    height: u32,
}
```