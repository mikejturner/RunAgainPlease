RunAgainPlease
==============

**RunAgainPlease** is a command-line tool which automates the process of running a series of calculations whilst varying a set of parameters of your choice.

##Installation

1. Download [**rap**](https://github.com/mikejturner/RunAgainPlease/blob/master/rap) and save it in a location in your `PATH` e.g. `/usr/local/bin`

2. Make `rap` executable with:

```Console
chmod u+x rap
```
##Usage

**rap** is run from the command-line and accepts one argument, a filename.

```Console
rap <filename>
```

**Example:**

```Console
rap gaussian.rap
```

The filename typically has a `.rap` extension.

## Structure of a `.rap` file

**rap** needs to know three things in order to do its job:

1. A list of parameters and the values they can take ([variables](#variables))
2. How to run your calculation (a [script](#script)).
3. A [template](#templates-and-substitutions) of the input file for a calculation.

The three sections are combined and put into a single `.rap` file. The [variables](#variables) and [script](#script) are specified by instructions to **rap** and must be prefixed with `@rap`. 

**Example:**

```
@rap var length 0.2,0.25,0.3,0.35,0.4
@rap script g09 < $rap_filename > $rap_basename.out
@rap script energy=`grep "SCF Done:" $rap_basename.out| token 5`
@rap script echo $length,$energy
@rap script rm $rap_basename.out
#T RHF/STO-3G

Title

0 1
H1
H2 H1 @rap:length
```

The above example runs a series of calculations using the [Gaussian](http://www.gaussian.com/) computation chemistry package. It defines one variable on the first line, then a four line script to run each calculation. The rest of the lines are a [template](#templates-and-subsitutions) of the Gaussian input file.

## Templates and Substitutions

All lines in the `.rap` that are *not* instructions to **rap** (nor [comments](Comments)) are assumed to be part of the template for the input file.

In the example [above](#structure-of-a-rap-file) the template is made up of the following lines:

```
#T RHF/STO-3G

Title

0 1
H1
H2 H1 @rap:length
```

For each calculation **rap** takes the template and substitutes the values of [variables](#variables) into it to produce a complete input file. The location of the substitutions is controlled by the placement of tokens in the template.

Tokens are defined using [variable](#variables) names:

```
@rap:<variable-name>
```

**Example:**

```
@rap:length
```

The token `@rap:length` will be replaced with the values of the variable `length`.

There is no restriction on where tokens can appear in the template and if the same token appears more than once then all occurrences get substituted. An alternative use of variables is discussed [here](#using-variables-as-command-line-arguments).

### Variables

Variables are defined using the `@rap var` instruction:

```
@rap var <variable-name> <variable-values>
```

Although variable names can be any combination of letters, numbers and punctuation, the following restrictions apply:

* No spaces.
* Don't use `@rap` or `@rap:` in the variable name.
* Don't call your variable `input`, `basename` or `extension`. ([Why?](#script-parameters))

Variable names can be as long as you like but try to keep them short but still descriptive e.g. `energy-cutoff` or `hydrogen_bond_length`

The variable-values part of the definition depends on what kind of variable you want. **rap** understands two basic kinds of variables: [Lists](#lists) and [Ranges](#ranges).

#### Lists

```
@rap var <variable-name> value1,value2,value3,...
```

Values in a list are comma-separated and can be text, numbers or a mixture.

**Example:**

```
@rap var noise moo,oink,woof
```

Here we are defining a variable called `noise` and will be given 3 values: `moo`, `oink` and `woof` when **rap** is run.

**Example:**

```
@rap var x 0.2,0.25,0.3,0.35,0.4
```

Here we are defining a variable called `length` and will be given 5 values from 0.2 up to 0.4 when **rap** is run.

#### Ranges

This variable type provides a way to specify ranges of numbers. They provide a convenient alternative to having to write out all the values when using lists. 

```
@rap var <variable-name> <start-value>:<step-size>:<number-of-steps>
```

Ranges can be use integers e.g. 1,2,3,... or real numbers e.g. 10.4,10.5,10.6,... as shown below.

**Example:**

```
@rap var x 1:2:5
```

Creates a variable `x` with the values `1`, `3`, `5`, `7` and `9`.

**Example:**

```
@rap var y 0.0:1.1:5
```

Creates a variable `y` with the values `0.0`, `1.1`, `2.2`, `3.3`, `4.4` and `5.5`.

If you know you want the range to begin at `start` and stop at `end` with `number-of-steps` steps, then the `step-size` can be calculated with:

```
step-size = (end - start) / (number-of-steps - 1)
```

#### File Substitution

A special variant of the list allows you to substitute the contents of other files into the [template](#templates-and-substitutions).

```
@rap var <variable-name> file:my-file-1,file:my-file-2,file:my-file-3,...
```

This is the same as an standard [list variable](#lists) except each file is prefixed by `file:`. The prefix is necessary to let **rap** know you want file substitution and not just text substitution. Files can be specified in the usual ways:

* Absolute path e.g. `/home/mike/data/methane.xyz`
* Relative path e.g. `data/methane.xyz`
* Or simply `methane.xyz` if this file is in the same directory that you will run **rap** in.

**Example:**

```
@rap var molecule file:data/methane.xyz,file:data/water.xyz,file:data/carbondioxide.xyz
```

This substitutes the contents of the three files `data/methane.xyz`, `data/water.xyz` and `data/carbondioxide.xyz` in place of the variable `molecule`.

#### Multiple Variables

If more than one variable is specified, **rap** will run calculations for all the unique combinations.

**Example:**

```
@rap  var  A  x,y,z
@rap  var  B  1,2,3
```

**rap** will run 9 calculations the first with `A=x,B=1` and the last with `A=z,B=3` as shown be the diagram below:

![Cartesian product example](http://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Cartesian_Product_qtl1.svg/220px-Cartesian_Product_qtl1.svg.png)

## Script

The script tells **rap** what steps need to be taken to perform a single calculation. Each step has a corresponding script instruction:

```
@rap script <bash-command>
```

The `bash-command` describes what happens in the step and is written in the [bash scripting language](http://mywiki.wooledge.org/BashGuide). It is beyond the scope of this document to explain bash scripting but some useful examples are given [below](#example-script-commands).

Behind the scenes **rap** combines all the script instructions together and creates a bash script that is used to run each calculation.

Here are some steps that you make want your script to perform:

* Setting up the conditions to successfully run the calculation (e.g. copy files, set environment variables etc.).
* Run the program that does the calculation (obviously).
* Extract useful numbers from the output of the calculation.
* Clean-up files that were generated in the course of running the calculation.
* Archiving the results of calculations.

### Script Parameters

**rap** automatically inserts a number of parameters into the script file that may be helpful when running your calculations. There are two types of script parameters: [variable-based](#variable-based-script-parameters) and [filename-based](#filename-based-script-parameters).

#### Variable-based Script Parameters

All the variables you created with `@rap var` are available in the script.

**Example:**

```
@rap var cutoff 1,2,3
@rap var element carbon,oxygen
```

The variables `cutoff` and `element` can be accessed in the script using the parameters `$cutoff` and `$element` respectively.

Having the variables available in the script is particular useful in the following situations:

* Using variables as [command line arguments](#using-variables-as-command-line-arguments)
* Storing the results of the calculation with the variables that produced it ([example](#printing-out-the-interesting-output)).

#### Filename-based Script Parameters

When **rap** is about to run a calculation it substitutes the [variables](#variables) into the [template](#templates-and-substitutions) and stores the result in a file. The name of this file is assigned by **rap**  and is unique for each calculation. Information about the assigned filename is made available in three script parameters: `$rap_filename`, `$rap_basename` and `$rap_extension`.

**Example:**

If the input filename is `5.inp` then,

```
$rap_filename = 5.inp
$rap_basename = 5
$rap_extension = inp
```

Here are a few of situations when these parameters come in handy:

* When you want to create an output filename based on the input filename.
* You need to rename the input filename to meet the criteria of the program running in the calculation.
* You want to archive the input files.

### Using Variables as Command-line Arguments

As an alternative to [substituting variables](#templates-and-substitutions) into a template, the variables can also be used as command-line arguments.

**Example:**

```
@rap script mkdir $molecule
```

This creates a new folder whose name is the value stored in the variable `molecule`.

### Example Script Commands

To provide some inspiration for writing you own scripts, here are explanations of the script instructions used in the [example](#structure-of-a-rap-file) introduced earlier.

**Example:**

```
@rap script g09 < $rap_filename > $rap_basename.out
```

This script instruction runs our calculation using `g09`. The input for the calculation comes from the file given by `$rap_filename` which we know is a [shell parameter](#filename-based-script-parameters) containing the filename in which **rap** stores the input. We construct a filename based on `$rap_basename` with the extension `.out` and store the output of the calculation in that file.

**Example:**

```
@rap script energy=`grep "SCF Done:" $rap_basename.out| token 5`
```

Here we are extracting a value from the file `$rap_basename.out` and storing it in a new shell parameter called `energy`. This is achieved by finding (using `grep`) the line in the output which contains the text `SCF Done:`.

We only want the number that represents the energy so we use `token` to select the 5th "word" on the line.

Later on when we want to access the energy parameter we must remember to refer to it as `$energy`.

**Example:**

```
@rap script echo $length,$energy
```

Here we are using the `echo` command to print out a comma-separated list of values that interest us (in this case the `length` and `energy` variables). This can be saved into a [.csv file](http://en.wikipedia.org/wiki/Comma-separated_values) and conveniently read into a spreadsheet program for graphing.

**Example:**

```
@rap script rm $rap_basename.out
```

Here we are using the `rm` command to remove the output files and keep things tidy.

## Gotchas and Advice

Below is some helpful advice to help you avoid problems when running **rap**.

### Substitutions

* Don't use tokens inside files that you intend to subsitute into the template. **rap** does *not* perform variable substitution within these included files.
* There is no limit on the number of variables you can define. However if you had 5 variables each taking 4 values that would result in 4^5 or 1024 calculations. Depending on how long each calculation takes you could be waiting a very long time for it to finish. (See section on [multiple variables](#multiple-variables)).

### Naming Variables

* Don't include spaces.
* Don't begin with `@rap` or `@rap:`.
* Don't use the names `$rap_input`, `$rap_basename` or `$rap_extension`.

### Comments

To disable any **rap** instruction, simply add **#** at the beginning of the line.

**Example:**

```
#@rap var length 0.2,0.25,0.3,0.35,0.4
```

**rap** will now ignore the variable called `length`.

## Issues

Please feel free to raise issues here on github:

https://github.com/mikejturner/RunAgainPlease/issues

## Maintainers

* Mike Turner (https://github.com/mikejturner)

## Licence

RunAgainPlease is licenced under the [MIT Licence](https://github.com/mikejturner/RunAgainPlease/LICENCE.md).
