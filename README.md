# Python task
## Part1 of task
The script should create a stack from the template and wait until the stack status changes from "IN PROGRESS" to "COMPLETE" or "FAILED".  
You can find the detailed description of the parameters below:  
- action (create, update, delete);
- stack name;
- template file name (can based on the stack name);
- stack parameters;
- log level (info, debug, error; by default info);
- log file name (by default to stdout);  
## Solution
"stack_wrapper.py" in "part1" fodler  

<b># Part2 of task</b><br>
Add to the script the ability to work with stack sequences. This will avoid errors with manually managing a large set of stacks. The script should performe following steps:<br>
	○ read the structure of the stacks<br>
	○ validate the templates<br>
	○ determine which the parameters are missing in the structure<br>
	○ perform an action (create or delete) with the target stack and all dependencies if the above steps are completed successfully<br>
The stack structure is stored in a file (yaml) and  includes following:<br>
	○ stack names<br>
	○ template files (can based on stack name)<br>
	○ dependencies (the order of creation and deletion)<br>
parameters passed to the stack<br>
<b># Solution</b><br>
"stack_wrapper.py" in "part2" fodler<br>
