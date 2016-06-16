---
layout: post  
title: Bash Script  
date: 2016-06-03  
categories: blog  
tags: [Bash]  
description: Bash Script  

---

## Bash Script ##

### Scripting Security ###

All of the Script are beginning with content below:  

```bash
#!bin/bash
set -o nounset
set -o errexit  
```

To do this for avoiding two common problem:    

 - Referencing undefined variables.  
 - Ignore failing to execute cmds.  

> **Note:**  
> At `errexit` mode, although it can catch the errors effectively, it can't capture all of the failed cmd. In some cases, some of the failed cmd is unable to detect.   
  

### Use [[ ]] Replace [ ] ###

Use [[ ]] can avoid problems such as the file name extension of the exception. And it can bring a lot of improvement in grammar and also adds new features.  

The `variable` and `explanation`  

```bash
||  #logic `or`(only used in double [])
&&  #logic `and`(only used in double [])
<   #character string compare
-lt #digital compare
=   #character string eq
==  #by globbing string comparison(used only in double square brackets)
=~  #use regular expressions to perform string comparisons(used only in double square brackets)
-n  #retval NULL string
-z  #NULL character
-eq #digital equal
-ne #digital unequal
```

### Regular Expression / Globbing ###

Example:  

```bash
t="abc123"
[[ "$t" == abc* ]]         # true (globbing comparison)
[[ "$t" == "abc*" ]]       # false (literal comparison)
[[ "$t" =~ [abc]+[123]+ ]] # true (regular expression comparison)
[[ "$t" =~ "abc*" ]]       # false (literal comparison)
```

String comparison can also be used as Globbing to the `case` statement:  

```bash
case $t in
abc*)  <action> ;;
esac  
```

### String Operations ###

1.Basic Operations  

```bash
f="path1/path2/file.ext"  

len="${#f}"        # = 20 (length of a string) 
	
# slicing operation: ${<var>:<start>} or ${<var>:<start>:<length>}
slice1="${f:6}"    # = "path2/file.ext"
slice2="${f:6:5}"  # = "path2"
slice3="${f: -8}"  # = "file.ext"(Note：front of "-" is a blank space)
pos=6
len=5
slice4="${f:${pos}:${len}}" # = "path2"
```

2.Replacement Operation  


```bash
f="path1/path2/file.ext"  
	
single_subst="${f/path?/x}"   # = "x/path2/file.ext"
global_subst="${f//path?/x}"  # = "x/x/file.ext" 
	
# splitting strings
readonly DIR_SEP="/"
array=(${f//${DIR_SEP}/ })
second_dir="${arrray[1]}"     # = path2
```
3.Remove the head or tail  

```bash
f="path1/path2/file.ext" 
	
# remove the head of the string.
extension="${f#*.}"  # = "ext" 
	
# remove the head of string through greedy matching.
filename="${f##*/}"  # = "file.ext" 
	
# remove the tail of the string.
dirname="${f%/*}"    # = "path1/path2" 

# remove the tail of string through greedy matching.
root="${f%%/*}"      # = "path1"   
```

### Build-in Variable ###

The `variable` and `explanation`

```bash
$0 #the name of script  
$n #the n-th variable of deliveriing to script
$$ #the PID of script
$! #the PID of previous executed-CMD  
$? #the state of the previous CMD exited
$# #the number of variable dliveried to script
$@ #the all of variable dliveried to script
$* #the all of variable of dliveried to script(as a character string)   
```   
  
### Debug ###

1.Grammar Check to the Script  

```bash
bash -n myscript.sh
```

2.Trace the executing cmd in Script  

```bash
bash -v myscript.sh 
```

3.Trace the executing cmd and extend the information in Script  

```bash
bash -x myscript.sh
```

> **Note:**  
> You can use `set -o verbose` and `set -o xtrace` in front of the script to assign `-v` `-x` eternally.  

### When do you not need to use script ###

- The script is too long, up to hundreds of lines.
- You need more complex than an array data structure.
- Escape with complex problems.
- Too much character string to dispose.
- Don't need to interact with other applications through pipe.
- Worry about performance.






