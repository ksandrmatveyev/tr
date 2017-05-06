# Python task
### Part2 of task
Add to the script the ability to work with stack sequences. This will avoid errors with manually managing a large set of stacks. The script should performe following steps:  
- read the structure of the stacks
- validate the templates
- determine which the parameters are missing in the structure
- perform an action (create or delete) with the target stack and all dependencies if the above steps are completed successfully
The stack structure is stored in a file (yaml) and  includes following:  
- stack names
- template files (can based on stack name)
- dependencies (the order of creation and deletion)
- parameters passed to the stack
### Solution
1. Cloudformation templates `*.json` from [salstack task](aws_training/final/cloudformation-stacks/README.md) for testing (in "examples" folder only a few templates)
2. `config.yaml`, which include:
   - `name` - stack names
   - `template` - file paths to cloudformation templates (current directory)
   - `require` - dependency from other stack (`~` - no dependency)
   - `parameters` - parameter list of key-value pairs (~ - no parameters)
3. `stack_wrapper.py`, which include:
   - subparsers, which allow using defaults parameters such as functions. But we can't use them with  [parent's parsers](https://docs.python.org/3/library/argparse.html#parents) (get heap of cli parameters). Has arguments below:
     - `--config` with default values as `config.yaml` from current directory
     - for `--log`  for setting log level to config. Must use `getattr()` function inside `main()` fucntion, otherwise get error. Also used predefined variants for log level
     - `logfile` with None(STDOUT) as default value  
  **Note:** _help stdout. (if we don't use any cli parameter, we get error without that)_
   - `open_file` function, which try open and readtemplate file. If OK, close file and return it, otherwise get exception. 
   - `stack_exists` function,which get response from AWS if stack exists. Needed for `delete_stack` function, which show us no error, if stack doesn't exists
   - `set_waiter` function, which try to create waiter and waite for AWS response, otherwise exception
   - `create_stack` function with parameters, which reads valid template file and creates stack with capabilities (hardcoded). Then set waiter using set_waiter function
   - `updade_stack` function with parameters, which reads valid template file and updates stack with capabilities (hardcoded). Then set waiter using set_waiter function
   - `delete_stack` function with parameters, which checks if stack exists and deletes stack. Then set waiter using set_waiter function
   - `main` function as entry point, where we get arguments fom parsers, configure logging and handle all exceptions from stack_exists, create_stack, updade_stack, delete_stack
   - `if __name__ == '__main__'`, which run main function  
### Using examples
Windows: `python stack_wrapper.py create-stack StackName --config config.yaml --log INFO --logfile log.log`  
Linux: `./stack_wrapper.py create-stack StackName --config config.yaml --log INFO --logfile log.log`  
#### Note:
Don't handle parameters for template (try to handle the in part2 of this task) and don't handle template file name, which can based on stack name (leave an opportunity of setting different names)  
Improvements since last commit:  
- handled almost all exceptions from functions in main function
- removed checking of log level and log file arguments. They are not needed
- added dictionary for waiters and constants for logging and stack functions
