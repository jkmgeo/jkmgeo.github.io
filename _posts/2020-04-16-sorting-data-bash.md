---
title: 'Sorting numerical data via shell script'
date: 2020-04-16
permalink: '/posts/2020/04/sorting-numerical-data-shell-script/'
tags:
    - bash
    - awk
    - sort
    - gmt
---

## Intro

Recently, I had to make a map in GMT, plotting the locations of various earthquakes, with the plot symbols colored by earthquake focal depth, and symbol size scaled to earthquake magnitude. Then went well, but the data were plotted in the order they were stored in the earthquake catalog file. This meant large earthquakes had large symbols, and plotted atop (and thus obscuring) chronologically earlier smaller earthquakes. But how to sort the data so the largest data were read first? Cue scratching the surface of `awk` and `sort` in my bash shell.

To begin, here's some sample data from the `eq.txt` file:

```
time,latitude,longitude,depth,magnitude,status
2020-01-01T00:00:01Z,34,-117,6.2,4.8,reviewed
2020-01-01T01:00:00Z,20,-100,4.3,7.7,reviewed
2020-03-03T12:00:00Z,14,-105,5,6.1,reviewed
2020-03-02T13:00:00Z,40,93,10,5.0,reviewed
```

Note that I assume that you're using `bash` from within the same directory as where your datafile (here, `eq.txt`) is stored. If not, `cd` over there. Any commands here to export an output file (e.g., `> output.txt`) will put the output file in the same directory.

Now, to plot the data, 4 columns are needed, representing `(x, y, c, s)` or x position, y position, color, and size. This order is specified by GMT's `psxy` plotting function, which I was using to make the map. So, how to extract the `longitude`, `latitude`, `depth`, and `magnitude` columns, with the rows sorted on the `magnitude` column for plotting?

## Using `awk`

First, we need to extract the data from the file, so we use `awk`, which is a shell utility. `awk` commands are formatted like: `awk 'action' <file> > <output>`. Here, that means using:

`awk -F, 'print' eq.txt > output.txt`,

where `-F,` tells `awk` that the `F`ield delimiter is a `,`.

But, this still isn't quite what I was looking for because, among other things, there's a header line I want to keep, and we don't want all the columns; just `(x, y, c, s)`. The `awk` command needs to be updated to:

`awk -F, 'NR==1 {print($3, $2, $4, $5)}; NR>1 {print($3, $2, $4, $5)}' eq.txt > output.txt`

All that extra stuff did two things:

1. `awk` looked at the record (or row) number (`NR`) if it was the first -- i.e., header -- line, it entered the `{print...}` action before the `;`. If `NR>1`, it proceeded to the second `{print...}` action.
2. In this case, the actions for both the header row and the non-header rows (`NR>1`) was the same: `print` out `$3, $2, $4, $5`... that is, the 3rd, 2nd, 4th, and 5th columns, in that order. These, if you look at `eq.txt` are `longitude`, `latitude`, `depth`, `magnitude`.

So, we're most of the way there! But we still need to sort the data on `magnitude`.

## Using `sort`

`sort` is another shell utility, like `awk`. Perhaps unsurprisingly, it sorts data. We can combine the output from `awk` into a `sort` command by "piping" (`|`) the `awk` output. Let's do that, and send the sorted output to a text file called `quakes_rsort.txt` so that we can review it and make sure the sorting worked properly.

`awk -F, 'NR==1 {print($3, $2, $4, $5)}; NR>1 {print($3, $2, $4, $5) | "sort -k4 -r -n"}' eq.txt > quakes_rsort.txt`

Here, for the lines that aren't the header, the `awk` output is piped into `sort`. This is why `sort -k4 -r -n` is in double quotes, and inside the brackets (`{ }`) of the `awk` command where actions are done. Now, a few notes about `sort` syntax: `-k` is used to specify the field (i.e., column) used to sort on. So, `-k4` sorts on the 4th column. But wait, isn't `magnitude` the **5th** column of our data! Well, yes, but remember that `sort` is taking the piped `awk` output, where we only extracted columns `$3, $2, $4, $5`, so `magnitude` is now actually column 4. I've specified `-n` to tell `sort` that this is numeric data (just to make sure there's no issues like you sometimes find with `ls` about files being named `1.txt`, `10.txt`, `11.txt`, ... `2.txt`, `20.txt`, ...) and, crucially, `-r`. The `-r` tells `sort` to do so in reverse (i.e., descending) fashion.

## Finishing up

So, putting it all together (with a modification to take the `sort` output and pipe it straight into `gmt psxy`):

`awk -F, 'NR==1 {print($3, $2, $4, $5)}; NR>1 {print($3, $2, $4, $5) | "sort -k4 -r -n"}' eq.txt | psxy $rgn $proj -Sc -i0:2,3+s0.01 -Ceqz -W.1,black -O -K >> $outfile.ps`

Now we have reversely sorted data, and plotted it on a map in GMT!

For the curious, those other commands specify that the plotted data is:
- defined by `$rgn` region, ...
- with `$proj` projection, ...
- using circles for symbols (`-Sc`), ...
- setting column parameters (`-i0:2,3+s0.01` leaves cols 0-2 -- i.e., lon, lat, depth; and yes, it's confusing that GMT indexes from 0 and `awk` from 1 -- alone and scales the values in column 3 by 0.01 so that the size of the earthquakes on the map aren't huge), ...
- plotting the symbols with a color scale defined by a palette named "eqz" (`-Ceqz`), ...
- with symbols outlined in black, 0.1 pt edges (`-W0.1,black`), ...
- laid overtop (`-O`) ...
- of my pre-existing postscript image (`$outfile.ps`) ...
- and leaving the file open for continued editing (`-K`).