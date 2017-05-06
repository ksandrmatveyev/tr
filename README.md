# Python task
### Part1 of task
The script should create a stack from the template and wait until the stack status changes from "IN PROGRESS" to "COMPLETE" or "FAILED".  
You can find the detailed description of the parameters below:  
- action (create, update, delete);
- stack name;
- template file name (can based on the stack name);
- stack parameters;
- log level (info, debug, error; by default info);
- log file name (by default to stdout);  
### Solution
"stack_wrapper.py" in "part1" fodler  
Short brief, that show what I use:  
- cloudformation templates for testing
- subparsers. They allow using defaults parameters such as functions. But we can't use them with  [parent's parsers](https://docs.python.org/3/library/argparse.html#parents) (get heap of cli parameters)
 - for `--log` must use `getattr()` for setting log level to config, otherwise get error
 - also used predefined variants for `--log`
- help stdout. (if we don't use any cli parameter, we get error without that)
- `open_file` function, which try open and readtemplate file. If OK, close file and return it, otherwise get exception. 
- `stack_exists` function,which get response from AWS if stack exists. Needed for `delete_stack` function, which show us no error, if stack doesn't exists
- `set_waiter` function, which try to create waiter and waite for AWS response, otherwise exception
- `create_stack` function with parameters, which reads valid template file and creates stack with capabilities (hardcoded). Then set waiter using set_waiter function
- `updade_stack` function with parameters, which reads valid template file and updates stack with capabilities (hardcoded). Then set waiter using set_waiter function
- `delete_stack` function with parameters, which checks if stack exists and deletes stack. Then set waiter using set_waiter function
- `main` function as entry point, where we get arguments fom parsers, configure logging and handle all exceptions from stack_exists, create_stack, updade_stack, delete_stack
- `if __name__ == '__main__'`, which run main function  
### Using examples
Windows: `stack_wrapper.py StackName TemplatePath.json --log INFO --logfile log.log`  
Linux: `./stack_wrapper.py StackName TemplatePath.json --log INFO --logfile log.log`  
#### Note:
Don't handle parameters for template (try to handle the in part2 of this task) and don't handle template file name, which can based on stack name (leave an opportunity of setting different names)  
Improvements since last commit:  
- handled almost all exceptions from functions in main function
- removed checking of log level and log file arguments. They are not needed
- added dictionary for waiters and constants for logging and stack functions
 
