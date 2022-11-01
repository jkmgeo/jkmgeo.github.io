---
layout: page
title: "TIL bash: clutter-free startup of Jupyter Lab in a terminal"
---


# `bash`: jupyter lab


### Context
When running `jupyter lab` in a bash terminal, lots of output gets printed at the outset, and checkpoints trigger yet more output to display to the terminal window. I found this annoying, not least because it inhibits using that same terminal window for other tasks. Cue trying to find a fix.


### Code
`nohup jupyter lab --no-browser &` + `<cr> <cr>`


### Result
This creates a file `nohup.out` in the working directory, and otherwise suppresses terminal output. The PID of the jupyter lab instance will be shown as output, however, which is useful because it can then be ended via:

`kill -9 <pid>`.

To open jupyter lab in a browser once initialized, use:

`jupyter lab list`

and click the link to the currently running server of your choice.


### Citation
Unfortunately, I'm not sure where I read this, other than StackOverflow.


<br>
<br>
[<< go back](./index.md)
