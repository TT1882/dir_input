# dir_input
A basic library to handle directory inputs in rust for linux.

## Functions
- get_dir_input

## Usage:
```rust
get_dir_input(message);
```
Where message is what the user will be asked.

## How it works

```rust
pub fn get_dir_input(message: &str) -> String {
    println!("{}", message);
    let home = home_dir().unwrap();
    let homeDir = home.display();  
    let mut userInput = String::new();
    let mut userInput2 = String::new();
    let mut lastPos: usize;
    let mut dirTest: bool;

    loop {
        // Gets user input for dir
        io::stdin().read_line(&mut userInput).expect("Failed to read line");

        lastPos = userInput.len()-1;

        // Dir format error handling
        if userInput.contains("~") {
            // If first character of string is a tilda
            if userInput.chars().next().unwrap() == '~' {
                userInput.remove(0);
                userInput = format!("{}{}", homeDir, userInput)
            }
            else {
                println!("Invalid location of tilda! Must be at the start of the directory!\n");
                get_dir_input(message);
            }
        }
        else if userInput.contains("$") {
            println!("Please do not use a environmental variable in the directory\n");
            get_dir_input(message);
        }
        else if &userInput[..lastPos] == "/" {
            userInput.pop();
        }
        
        // Check if dir exists
        dirTest = Path::new(&userInput.trim()).exists();

        // If directory does not exist
        while dirTest == false{
            println!("Directory does not exist, would you like to create it? yn exit");
            io::stdin().read_line(&mut userInput2).expect("Failed to read line");

            if userInput2.trim().to_lowercase().to_string() == "y" {
                // Creates directory
                userInput = userInput.trim().to_string();
                run_cmd!(mkdir -p $userInput).expect("Dir not valid or needs superuser privileges to access")
            }
            else if userInput2.trim().to_lowercase().to_string() == "exit" {
                exit(1)
            }
            else {
                get_dir_input(message);
            }
            dirTest = Path::new(&userInput.trim()).exists();
        } 
        return userInput.trim().to_string();
    }
}
```

The message specified is displayed and the user gives an input. The input is altered so it works with GNU tools and tildas are expanded to full directories (~ = /home/user/). If the directory does not exist, the user is propted if they want to create it. 
