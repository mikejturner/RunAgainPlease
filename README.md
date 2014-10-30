RunAgainPlease
==============

**RunAgainPlease** is a commandline tool which automates the process of running a series of calculations whilst varying a set of parameters of your choice.

##Installation

1. Download [**rap**](https://github.com/mikejturner/RunAgainPlease/blob/master/rap) and save it in a location in your `PATH` e.g. `/usr/local/bin`

2. Make `rap` executable with:

```Console
chmod u+x rap
```
##Usage

**rap** is run from the commandline and accepts one argument.

```Console
rap <filename>
```

Example:

```Console
rap gaussian.rap
```

## Structure of a `.rap` file

**rap** needs to know three things in order to do its job:

1. A list of parameters and the values they can take ([variables](#variables))
2. How to run your calculation (a [script](#script)).
3. A [template](#templates-and-substitutions) of the input file for a calculation.

The three sections are combined and put into a single `.rap` file. The [variables](#variables) and [script](#script) are specified by instructions to **rap** and must be prefixed with `@rap`. 

Example:

```
@rap var length 0.2,0.25,0.3,0.35,0.4
@rap script g09 < $input > $basename.out
@rap script energy=`grep "SCF Done:" $basename.out| token 5`
@rap script echo $length,$energy
@rap script rm $basename.out
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

Example:

```
@rap:length
```

The token `@rap:length` will be replaced with the values of the variable `length`. If there is no instruction defining the variable with the name `length` in the `.rap` file then no substitution will take place i.e. the token will be simply ignored. This will probably break your calculation because the program performing the calculation will have no idea what `@rap:length` means.

There is no restriction on where tokens can appear in the template and if the same token appears more than once then all occurrences get substituted.

### Variables

Variables are defined using the `@rap var` instruction:

```
@rap var <variable-name> <variable-values>
```

Although variable names can be any combination of letters, numbers and punctuation, the following restrictions apply:

* No spaces.
* Don't use `@rap` or `@rap:` in the variable name.
* Don't call your variable `input`, `basename` or `extension`. ([Why?](#script-variables))

Variable names can be as long as you like but try to keep them short but still descriptive e.g. `energy-cutoff` or `hydrogen_bond_length`

The variable-values part of the definition depends on what kind of variable you want. **rap** understands two basic kinds of variables: [Lists](#lists) and [Ranges](#ranges).

#### Lists

```
@rap var <variable-name> value1,value2,value3,...
```

Values in a list are comma-separated and can be text, numbers or a mixture.

Example:

```
@rapvar noise moo,oink,woof
```

Here we are defining a variable called `noise` and will be given 3 values: `moo`, `oink` and `woof` when **rap** is run.

Example:

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

Example:

```
@rap var x 1:2:5
```

Creates a variable `x` with the values `1`, `3`, `5`, `7` and `9`.

Example:

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

Example:

```
@rap var molecule file:data/methane.xyz,file:data/water.xyz,file:data/carbondioxide.xyz
```

This substitutes the contents of the three files `data/methane.xyz`, `data/water.xyz` and `data/carbondioxide.xyz` in place of the variable `molecule`.

#### Multiple Variables

If more than one variable is specified, **rap** will run calculations for all the unique combinations.

Example:

```
@rap  var  A  x,y,z
@rap  var  B  1,2,3
```

**rap** will run 9 calculations the first with `A=x,B=1` and the last with `A=z,B=3` as shown be the diagram below:

![Cartesian product example](http://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Cartesian_Product_qtl1.svg/220px-Cartesian_Product_qtl1.svg.png)

## Script

The script tells **rap** how to run the calculation and is generated from one or more script instructions of the form:

```
@rap script <bash-command>
```

**rap** automatically turns the script instructions into a [bash](http://en.wikipedia.org/wiki/Bash_(Unix_shell)) script.

There are many tutorials on the web for learning [bash scripting](http://mywiki.wooledge.org/BashGuide).

The most important command is the one that runs the calculation. It depends on what program you are using for your calculations.

### Script Variables

In order to make writing scripts easier **rap** adds a number of variables into the script. These script variables fall into two types...

When **rap** is about to run a calculation it substitutes the [variables](#variables) into the [template](#templates-and-substitutions) and saves the input into a file. What is the filename of the input? **rap** creates a unique filename for each calculation. 

Always available `$input`, `$basename` and `$extension`

All parameters are available in the script.

Example:

```
@rap var cutoff 1,2,3
```

is available in the script as `$cutoff`.

### Example Script Commands

#### Running the program

```
@rap script g09 < $input > $basename.out
```

#### Extracting interesting output

```
@rap script energy=`grep "SCF Done:" $basename.out| token 5`
```

#### Printing out the interesting output

```
@rap script echo $length,$energy
```

#### Tidying up

```
@rap script rm $basename.out
```


## Gotchas and Advice

:exclamation: Tokens are *not* substituted within included files

:exclamation: There is no limit on the number of variables you can define. However if you had 5 variables each taking 4 values that would result in 4^5 or 1024 calculations. Depending on how long each calculation takes you could be waiting a very long time for it to finish. (See section on [multiple variables](#multiple-variables)).

### Naming Variables

* Don't begin with `@rap`.
* Don't call them `$input`, `$basename` or `$extension`.

### Comments

To disable any **rap** instruction, simply add **#** at the beginning of the line.

Example:

```
#@rap var length 0.2,0.25,0.3,0.35,0.4
```

## Issues

Please feel free to raise issues here on github:

https://github.com/mikejturner/RunAgainPlease/issues

## Maintainers

* Mike Turner (https://github.com/mikejturner)

## Licence

[MIT Licence](https://github.com/mikejturner/RunAgainPlease/LICENCE.md)
