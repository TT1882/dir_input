# dir_input
A basic crate to handle directory inputs in rust for linux.

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
    let home_dir = home.display();  
    let mut user_input = String::new();
    let mut user_input2 = String::new();
    let last_pos: usize;
    let mut dir_test: bool;

    loop {
        // Gets user input for dir
        io::stdin().read_line(&mut user_input).expect("Failed to read line");

        last_pos = user_input.len()-1;

        // Dir format error handling
        if user_input.contains("~") {
            // If first character of string is a tilda
            if user_input.chars().next().unwrap() == '~' {
                user_input.remove(0);
                user_input = format!("{}{}", home_dir, user_input)
            }
            else {
                println!("Invalid location of tilda! Must be at the start of the directory!\n");
                get_dir_input(message);
            }
        }
        else if user_input.contains("$") {
            println!("Please do not use a environmental variable in the directory\n");
            get_dir_input(message);
        }
        else if &user_input[..last_pos] == "/" {
            user_input.pop();
        }
        
        // Check if dir exists
        dir_test = Path::new(&user_input.trim()).exists();

        // If directory does not exist
        while dir_test == false{
            println!("Directory does not exist, would you like to create it? yn exit");
            io::stdin().read_line(&mut user_input2).expect("Failed to read line");

            if user_input2.trim().to_lowercase().to_string() == "y" {
                // Creates directory
                user_input = user_input.trim().to_string();
                //run_cmd!(mkdir -p $user_input).expect("Dir not valid or needs superuser privileges to access")
                Command::new("mkdir")
                    .args(["-p", user_input.as_str()]).output().expect("Dir not valid or needs superuser privileges to access");
            }
            else if user_input2.trim().to_lowercase().to_string() == "exit" {
                exit(1)
            }
            else {
                get_dir_input(message);
            }
            dir_test = Path::new(&user_input.trim()).exists();
        } 
        return user_input.trim().to_string();
    }
} 
```
The message specified is displayed and the user gives an input. The input is altered so it works with GNU tools and tildas are expanded to full directories (~ = /home/user/). If the directory does not exist, the user is propted if they want to create it. 