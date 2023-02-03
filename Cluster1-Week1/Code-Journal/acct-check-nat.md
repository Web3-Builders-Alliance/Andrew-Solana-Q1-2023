## Code journal for [Acct Check NAT](https://github.com/solana-developers/program-examples/blob/main/basics/checking-accounts/native/program/src/lib.rss)  <br> <br>

### What are the concepts (borrowing, ownership, vectors etc)?
The concepts utilized in the code are structs, iterators, if statement,  next_account_info convience function, heavy utilazion of the built in ProgramError's that program_error::ProgramError provides.   <br> <br>

### What is the organization?
 One simple function that takes in a program_id, a struct of accounts and ix data  <br> <br>

### What is the contract doing? What is the mechanism? 
Contract is doing various checks to ensure that the passed via the accounts are passing implimented checks  <br> <br>

### How could it be better? More efficient? Safer? 
Pretty simple implimentation so I think this is probably as safe as it gets. 
  
<br>
------
<br>

```rust
use solana_program::{
    account_info::{ AccountInfo, next_account_info },
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program_error::ProgramError,
    pubkey::Pubkey,
    system_program,
};

entrypoint!(process_instruction);

fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    _instruction_data: &[u8]
) -> ProgramResult {
    // Ensure that the program id passed in is the actual program id using the check_id method on solana_program
    if system_program::check_id(program_id) {
        return Err(ProgramError::IncorrectProgramId); // #[error("A signature was required but not found")]
    }

    // Ensure that the instruction has 4 accounts passed in by checking array length is greater than 4
    if accounts.len() < 4 {
        msg!("This instruction requires 4 accounts:");
        msg!("  payer, account_to_create, account_to_change, system_program");
        return Err(ProgramError::NotEnoughAccountKeys); // #[error("Failed to borrow a reference to account data, already borrowed")]
    }

    // Convert accounts into an interable and use next_account_info to step through the accounts to parse them out into respective varables
    let accounts_iter = &mut accounts.iter();
    let _payer = next_account_info(accounts_iter)?;
    let account_to_create = next_account_info(accounts_iter)?;
    let account_to_change = next_account_info(accounts_iter)?;
    let system_program = next_account_info(accounts_iter)?;

    // Verify that the account_to_create hasn't already been initalized by checking if it contains lamports greater than 0
    msg!("New account: {}", account_to_create.key);
    if account_to_create.lamports() != 0 {
        msg!("The program expected the account to create to not yet be initialized.");
        return Err(ProgramError::AccountAlreadyInitialized); // #[error("An attempt to operate on an account that hasn't been initialized")]
    }

    // Same as above but checking if account hasn't been initalized
    msg!("Account to change: {}", account_to_change.key);
    if account_to_change.lamports() == 0 {
        msg!("The program expected the account to change to be initialized.");
        return Err(ProgramError::UninitializedAccount); // #[error("The instruction expected additional account keys")]
    }

    // Check if the program owner is not the program id
    if account_to_change.owner != program_id {
        msg!("Account to change does not have the correct program id.");
        return Err(ProgramError::IncorrectProgramId); // #[error("A signature was required but not found")]
    }

    // Check if system_program keep isn't the system_program that has been passed into AccountInfo struct 
    if system_program.key != &system_program::ID {
        return Err(ProgramError::IncorrectProgramId); // #[error("A signature was required but not found")]
    }

    Ok(())
}

```