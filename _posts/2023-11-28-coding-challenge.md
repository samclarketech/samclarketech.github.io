---
layout: post
title: A Coding Challenge
---

### The Brief
Given five Word documents (CVs), use a programming language of your choice to find which files contain the word "Engineer". Return the names of these files.

![The Word Documents](/assets/images/word-docs.PNG)

### The Solution
I wrote the below solution in Bash. It is not elegant or efficient, but it works! The script needs to be in the parent directory of a directory named "working" which contains the Word docs.

Firstly, a for loop is used to iterate over each file and the filename is stored in a variable.

Given Office documents are ZIP files, I use the unzip command to drop the contents of each to a unique folder. I then use grep to recursively search the unzipped files for the word "engineer" and, if found, the file name is printed to the screen.

Any unwanted output is sent to /dev/null.

Script link: [https://github.com/samclarketech/coding-challenge](https://github.com/samclarketech/coding-challenge)

```bash
#!/bin/bash

# thought process/notes:

# iterate over the files in the containing dir
# take the filename and store in variable
# Brian\ Morris\ CV.docx - filename with escaped spaces
# make a unique dir
# unzip the docx to the unique dir
# grep contents of unique dir for "Engineer"
# if found return the filename
# send all unwanted output to /dev/null so that only the file names are returned
# unzip error to fix:
# unzip:  cannot find or open working/'Brian, working/'Brian.zip or working/'Brian.ZIP.

directory=working/*
dir_name=working

for file in $directory
do
	# split filename by the space, store each part in var
	filename1=$(echo $file | awk -F '/' '{print $2}' | cut -d' ' -f1)
	filename2=$(echo $file | awk -F '/' '{print $2}' | cut -d' ' -f2)
	filename3=$(echo $file | awk -F '/' '{print $2}' | cut -d' ' -f3)

	unique_dir=$(echo $file | awk -F '/' '{print $2}' | cut -d' ' -f1)

	filename=$filename1' '$filename2' '$filename3

	unzip $dir_name/$filename1\ $filename2\ $filename3 -d $dir_name/$unique_dir >/dev/null
	
	if grep -ri "engineer" $dir_name/$unique_dir >/dev/null
	then
		echo $filename
	fi
done
```

#### Output
![The output](/assets/images/solution.PNG)
