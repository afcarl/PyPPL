# Command line argument parser
<!-- toc -->

This module is just for your convenience. You are free to use Python's built-in `argparse` module or just use `sys.argv`.

To start with, just import it from `PyPPL`:
```python
from pyppl import params
```

## Add an option
```python
params.opt = 'a'
# or
# params.opt.setValue('a')
params.opt2.setType('list').setRequired()

# then don't forget to parse the command line arguments:
params.parse()
```
Then `python pipeline.py -opt b -opt2 1 2 3` will overwrite the value.  
`list` option can also be specified as `-opt2 1 -opt2 2 -opt2 3`
To use the value:
```python
var = params.opt.value + '2'
# var is 'b2'
var2 = params.opt2.value + [4]
# var2 is [1, 2, 3, 4]
```

If you feel annoying using `params.xxx.value` to have the value of the option, you can convert the `params` object to a `Box` object (inherited from `OrderedDict`):
```python
params.parse()
ps = params.asDict()
# or just
# ps = params.parse()
```
Then you can use `params.xxx` to get the value directly:
```python
var = ps.opt + '2'
var2 = ps.opt2 + [4]
```


## Set attributes of an option
An option has server properties:
- `desc`: The description of the option, shows in the help page. Default: `[]`
- `required`: Whether the option is required. Default: `False`
- `show`: Whether this option should be shown in the help page. Default: `True`
- `type`: The type of the option value. Default: `str`
- `name`: The name of the option, then in command line, `-<name>` refers to the option.
- `value`: The default value of the option.

You can either set the value of an option by 
```python
params.opt.required = True
```
or
```python
params.opt.setRequired(True)
# params.opt.setDesc('Desctipion')
# params.opt.setShow(False)
# params.opt.setType(list)
# params.opt.setName('opt1')
# params.opt.setValue([1,2,3])
# or they can be chained:
# params.opt.setDesc('Desctipion') \
#           .setShow(False)        \
#           .setType(list)         \
#           .setName('opt1')       \
#           .setValue([1,2,3])
```

### About `type`
#### Declear the type of an option
You can use either the type itself or the type name:  
`params.opt.type = int` or `params.opt.type = 'int'`

#### Infer type when option initialized
When you initialize an option:
```python
# with nothing specified
params.opt
# with a default value
params.opt = 1
```
The type will be inferred from the value. In the first case, the type is `None`, will in the second, it's `'int'`

!!! note

    Let initialize an option first: `params.opt`, then when the value is set explictly:  
    `params.opt.value = "a"`  
    Or when the value replaced (set value after initialization):  
    `params.opt = "a"`  
    The type will not be changed (it's `None` in this case).

#### Allowed types
Literally, allowed types are `str`, `int`, `float`, `bool` and `list`. But we allow subtypes (types of elements) for `list`. By default, the value of `list` elements will be recognized automatically. For example, `'1'` will be recognized as an integer `1`, and `"True"` will be recognized as bool value `True`. You can avoid this by specified the subtypes explictly: `params.opt.type = 'list:str'`, then `'1'` will be kept as `'1'` rather than `1`, and `"True"` will be `"True"` instead of `True`.

You can use shortcuts for the types:
```
i     -> 'int'
f     -> 'float'
b     -> 'bool'
s     -> 'str'
l     -> 'list'
array -> 'list'
```
For subtypes, you can also do `params.opt.type = 'l:s'` indicating `list:str`. Or you can even mix them: `params.opt.type = 'l:str'` or `params.opt.type = 'list:s'`

#### Overwrite the type from command line
Even though we may declear the type of an option by `params.opt.type = 'list:str'`, the users are able to overwrite it by pass the type through command argument:  
`> program -opt:list:int 1 2 3`  
Then we will get: `params.opt.value == [1,2,3]` instead of `params.opt.value == ['1', '2', '3']` when we do:  
`> program -opt 1 2 3`  
Flexibly, we can have mixed types of elements in a list option:  
`> program -opt:list:bool 1 -opt:list:int 2 -opt:list:str 3`  
We will get: `params.opt.value == [True, 2, '3']`

## Load params from a dict
You can define a `dict`, and then load it to `params`:
```python
d2load = {
    'p1': 1,
    'p1.required': True,
    'p2': [1,2,3],
    'p2.show': True,
    'p3': 2.3,
    'p3.desc': 'The p3 params',
    'p3.required': True,
    'p3.show': True
}
params.loadDict (d2load)
# params.p1.value    == 1
# params.p1.required == True
# params.p1.show     == False
# params.p1.type     == int
# params.p2.value    == [1,2,3]
# params.p2.required == False
# params.p2.show     == True
# params.p2.type     == list
# params.p3.value    == 2.3
# params.p3.required == True
# params.p3.show     == False
# params.p3.type     == float
```
Note that by default, the options from the `dict` will be hidden from the help page. If you want to show some of them, just set `p2.show = True`, or you want to show them all: `params.loadDict(d2load, show = True)`

## Load params from a configuration file
If the configuration file has a name ending with '.json', then `json.load` will be used to load the file into a `dict`, and `params.loadDict` will be used to load it to `params`

Else python's `ConfigParse` will be used. All params in the configuration file should be under one section with whatever name:
```ini
[Config]
p1: 1
p1.required: True
p2: [1,2,3]
p2.show: True
p3: 2.3
p3.desc = 'The p3 params'
p3.required: True
p3.show: True
```

Similarly, you can set the default value for `show` property by: `params.loadCfgfile("params.config", show=True)`

!!! hint

    `params` is a singleton of `Parameters`. It's a combination of configuration and command line arguments. So you are able to load the configurations from files, which will be used as default values, before you parse the command line arguments. You are also able to choose some of the options for the users to pass value to, and some hidden from the users.

## Preseved option names
We have a certain convention of the option names used with `params`:
- Make sure it's only composed of alphabetics, underscores and hyphens.
- Make sure it starts with alphabetics.
- Make sure it's not one of these words (`help`, `loadDict`, `loadFile` and `asDict`)

## Set other attributes of params
### Show the usages/example/description of the program
`params('usage', ["{prog} -h", "{prog} -infile <infile> [options]"])`  
`params('example', ["{prog} -h", "{prog} -infile path/to/infile -out /path/to/outfile"])`  
`params('desc', ["This program does this.", "This program also does that."])`  
Multiple lines can also be passed as a string with `"\n"`:  
`params('desc', ["This program does this.\nThis program also does that."])`

### Set the prefix of option names
By default, the option names are recognized from the command line if they follow the convention and start with `-`. You may change it, let's say you want the option names to start with `--`:  
`params('prefix', '--')`  
The only `prog --opt 1` will be parsed, instead of `prog -opt 1` to get the value by `params.opt.value`  

!!! note

    You cannot use an empty prefix

### Set the default help options
By default, the help options are `['-h', '--help', '-H', '-?', '']`. The empty string (`''`) enables the program to print the help page without arguments be passed from the command line. If you don't want it, you can remove it from the default help options:  
`params('hopts', '-h, --help, -H, -?')` or  
`params('hopts', ['-h', '--help', '-H', '-?'])`  
Or you can just simply remove some of them by:  
`params('hopts', ['-H', '-?', ''], excl = True)`  
Then you will only have `['-h', '--help']`


!!! hint

    These statements can also be chained:  
    `params('usage', ["{prog} -h", "{prog} -infile <infile> [options]"])('prefix', '--')('hopts', ['-H', '-?', ''], excl = True)`