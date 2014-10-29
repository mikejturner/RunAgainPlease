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

For each calculation **rap** creates an input file and subsitutes values into it. The values come the from [variables](#variables) and the substitutions depend upon the locations of tokens embedded in the template of the input file.

Tokens are defined:

```
@rap:<variable-name>
```

Example:

```
@rap:foo
```

### Variables

Variables are defined using the `@rap var` instruction:

```
@rap var <variable-name> <variable-values>
```

**rap** understands two basic kinds of variables: [lists](#lists) and [ranges](#ranges).

#### Lists

The values are a comma-separated list:

```
@rap var <variable-name> value1,value2,value3,...
```
The values can be can be text or numbers.

Example:

```
@rapvar noise moo,oink,woof
```

Here we are defining a variable called `noise` and will be given 3 different values: `moo`, `oink` and `woof` when **rap** is run.

Example:

```
@rap var x 0.2,0.25,0.3,0.35,0.4
```

Here we are defining a variable called `length` and will be given 5 different values from 0.2 to 0.4 when **rap** is run.

#### Ranges

These allow a range of numeric values to be specified for a given parameter. The syntax is:

```
@rap var <variable-name> <start>:<step>:<number of values>
```

Example:

```
@rap var 1:2:5
```

Example:

```
@rap var 0.1:1.0:5
```

#### Substitution from files

```
@rap var <variable-name> file:filename1,file:filename2,file:filename3
```

Example:

```
@rap var molecule file:methane.xyz,file:water.xyz,file:carbondioxide.xyz
```

#### Multiple Variables

If more than one variable is specified, **rap** will run calculations for all combinations. The combinations are based on the [cartesian product](http://en.wikipedia.org/wiki/Cartesian_product).

Example:

```
@rap  var  A  x,y,z
@rap  var  B  1,2,3
```

**rap** will run 9 calculations the first with `A=x,B=1` and the last with `A=z,B=3` as shown be the diagram below:

![Cartesian product example](http://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Cartesian_Product_qtl1.svg/220px-Cartesian_Product_qtl1.svg.png)

## Script

### Script Variables

Always available `$input`, `$basename` and `$extension`

All parameters are available in the script.

Example:

```
@rap var cutoff 1,2,3
```

is available in the script as `$cutoff`.

## Comments

To disable any of the **rap** instructions, simply prefix the line with the **#** character.

Example:

```
#@rap var length 0.2,0.25,0.3,0.35,0.4
```

## Gotchas and Advice

:exclamation: Tokens are *not* substituted within included files

:exclamation: There is no limit on the number of variables you can define. However if you had 5 variables each taking 4 values that would result in 4^5 or 1024 calculations. Depending on how long each calculation takes you could be waiting a very long time for it to finish. (See section on [multiple variables](#multiple-variables)).

### Variable names

* Don't begin with `@rap`.
* Don't call them `$input`, `$basename` or `$extension`.

## Issues

Please feel free to raise issues here on github:

https://github.com/mikejturner/RunAgainPlease/issues

## Maintainers

* Mike Turner (https://github.com/mikejturner)

## Licence

[MIT Licence](https://github.com/mikejturner/RunAgainPlease/LICENCE.md)
