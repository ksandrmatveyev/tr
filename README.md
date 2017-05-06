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
     - `--log`  for setting log level. Must use `getattr()` function inside `main()` fucntion, otherwise get error. Also used predefined variants for log level
     - `--logfile` with None(STDOUT) as default value  
  **Note: added help stdout. (if we don't use any cli parameter, we get error without that)**
   - `open_file()` function, which try open and read template file. If OK, close file and return it, otherwise get exception and exit.  
  **Note: exit point 1**
   - `get_template_params()` fucntion, which validate template (using validate_template() function and read  template as parameter) and get parameters from dictionary by key `Parameters`. If default value doesn't exists, set empty value. Return new sorted list of key-value pairs (Parameter: value)
   - `get_config()` fucntion, which try open and read yaml config file. If OK, return read config, otherwise exception and exit.  
  **Note: exit point 2**
   - `match_parameters()` fucntion, which try to get template parameters (using `get_template_params()` fucntion) by key and search them in config file (if exists, use parameter value from config file). If OK, return list of dictionaries (key-value pairs), otherwise exception and exit.  
  **Note: exit point 3**
   - `stack_exists()` function, which get response from AWS if stack exists. Needed for action stack functions. Also, It allows us continue create/update from next stack, if previuos stack already exists or delete stack if it already exists. Return True/False
   - `set_waiter()` function, which try to create waiter and waite for AWS response, otherwise exception and exit.  
  **Note: exit point 4**
   - `get_dict_of_lists_dependency()` function, which returns dictionary of lists as value for each key from read config file (as fucntion parameter). Needed for creating some "chain" for deleting stacks
   - `resolve_create_dependencies()` fucntion, which returns list of stack dependencies chain from read config for creating of assigned stack by key 'require' (empty if value doesn't exist)
   - `resolve_delete_dependencies()` funcion, which returns list of stack dependencies chain from dictionary (which `get_dict_of_lists_dependency()` returns) for deleting of assigned stack
   - `create_stack` function, which creates dependent stacks consistently. If some stack already exists, continue creating with next. Details:
     - use get_config(),
     - resolve_create_dependencies(),
     - for loop
       - get template path by key 'template'
       - read template file by `open_file()`
       - get parameters by match_parameters()
       - reads valid template file and creates stack with
       - if some stack already exists, continue creating with next. 
         - set waiter using set_waiter()  

     **Note:** capabilities (hard coded for now)
   - `updade_stack` function, which updates dependent stacks consistently, if those stacks are already exist. Details:
     - use get_config(),
     - resolve_create_dependencies(),
     - for loop
       - get template path by key 'template'
       - read template file by `open_file()`
       - get parameters by match_parameters()
       - reads valid template file and creates stack with
       - if those stacks are already exist, updating them. 
         - set waiter using set_waiter()  

     **Note:** capabilities (hard coded for now)
   - `delete_stack` function, which deletes dependent stacks consistently, if those stacks are already exist. Details:
     - use get_config(),
     - get_dict_of_lists_dependency(),
     - resolve_delete_dependencies(),
     - for loop
       - if those stacks are already exist, updating them.
         - set waiter using set_waiter()
   - `main` function as entry point, where we get arguments fom parsers, configure logging and handle all exceptions from stack_exists, create_stack, updade_stack, delete_stack.  
**Note: exit point 5**
   - `if __name__ == '__main__'`, which run main function  
### Using examples
Windows: `python stack_wrapper.py create-stack StackName --config config.yaml --log INFO --logfile log.log`  
Linux: `./stack_wrapper.py create-stack StackName --config config.yaml --log INFO --logfile log.log`  
#### Note:
As you see above I don't handle capabilities and have dublicated logic inside create_Stack(), update_stack(), delete_Stack() functions. I'm working on that but I haven't had results for now :(
