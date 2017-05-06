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
"stack_wrapper.py" in "part2" fodler  
