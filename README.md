# Introduction:

Machinic-image has a 'boook' source named 'BYTEBOOOK' to use for testing. It generates a set of images with a booklike structure: a cover, table of contents, introduction with roman numerals, numbered pages with numbering appearing in corners and centers for new chapters. The source also stores marker positions of devices that can be incremented or decremented to simulate turning pages. This structure hopefully allows testing the entire system from slurping via hardware or software to defining the dss (using dss-ui) and then testing the queuing-ui and fold-ui for the mundane process of gathering and correctly arranging material. 

This tutorial should serve as an introduction to the machinic ecosystem and various concepts and provide an approach that can be seamlessly used with realworld booklike structures...

Currently the installation of the machinic ecosystem(machinic-core, machinic-image, various ma-* packages) is not covered. They are assumed to be installed, this will be covered in another tutorial...

# Ecosystem
Rough ecosystem outline and general philosophies:
* Encourage rapid and easy modification and prototyping
* Reduce friction wherever possible
* Loose, layered, moderately resilient, semi-structured, ecosystem
* Provide tools to feed modifications back into higher level specifications
* Generate code from high level specifications
* Provide high, mid- and low level tooling (especially to work with above points)
* Use DSLs as exploratory interfaces and provide as cli tools and packages
* Modular user interfaces that take advantage of shape and color, and use the same core tools as clis or other interfaces

# Getting Started:

_Let's see if BYTEBOOOK is running..._

Check if byteboook service is running using `ma-cli`. Ma-cli provides a variety of  interactive commandline interfaces: some novel and some wrappers that supply host and port information. The `nomad` command mostly wraps the hashicorp `nomad` command with some additional commands such as `restart` and `metalogs`.

```
ma-cli nomad
nomad:xxx.xxx.x.x:xxxx>status
```
should include:
```
ID                               Type     Priority  Status   Submit Date
byteboook                        service  50        running  03/09/18 13:17:50 UTC
```

# Simple and Sophisticated Processes

With a goal of gathering material from the byteboook source by incrementing positions and via some interaction and slurping via some interaction. A meta-goal of the tutorial(and the machinic ecosystem) is that the system for this process should translate seamlessly into other configurations so that the slurped position markers become separate cameras above some material.

A simple process might look like:
1. Simulate turning page(s)
2. Slurp
3. repeat 1 & 2 until some threshold
4. check slurped material

A more sophisticated process might be:
1. Simulate turning page(s)
2. slurp some representative pages from various positions in the structure
3. define domain specific structure using dss-ui
4. use queue-ui and fold-ui to observe progress
5. repeat 1 & 2 while doing 4

This tutorial will cover the longer process but it may be useful to briefly show how the simple process could be done using the generated permutations:

Start in the generated directory of machinic-image:

```
cd machinic-image/generated
ls
```

_Where do these files come from?_

`cat machinic-image/regenerate.sh`

The `machine.xml` is run through the codegen script from [ma](https://github.com/galencm/ma) that uses [gsl](https://github.com/zeromq/gsl) to generate a variety of files used by the machine and for user access. For example this allows an peripheral to be generated as a gui button and c code to be loaded on an esp82666 that both have the same output to the system.

The xml contains a variety of material that can be handled or ignored by various gsl passes.

For the tutorial, the relevant xml in `machinic-image/machine.xml` for buttons and boooks:

book and peripherals:

'Page' turning:

Messages are sent via mqtt(bridged to redis) or redis using pubsub. Redis pubsub is where routing is handled. The destination in the peripherals below begins with '/set/' which is a special prefix to set values( instead of just delivering message to channel).
```
    <peripheral type = "button" alternative_press = " " description = "increment pages, simulate turning pages towards end">
        <output type = "integer" value = "+= 2" destination = "/set/BYTEBOOOK/marker:capture1" />
        <output type="integer" value="+= 2" destination = "/set/BYTEBOOOK/marker:capture2" />
    </peripheral>

    <peripheral type = "button" alternative_press = " " description = "decrement pages, simulate turning pages towards beginning">
        <output type = "integer" value = "-= 2" destination = "/set/BYTEBOOOK/marker:capture1" />
        <output type="integer" value="-= 2" destination = "/set/BYTEBOOOK/marker:capture2" />
    </peripheral>
```

boook source:

'wireup' is used to wrap file/type with zerorpc

```
    <source type="primitive_bytes_indexable" source="boook" location="BYTEBOOOK" wireup = "true">
        <set peripheral="capture1" symbol="marker" value="-1" overwrite="true" />
        <set peripheral="capture2" symbol="marker" value="0" overwrite="true" />
    </source>
```

This may seem confusing, both to setup and while running, but hopefully offers advantages of decoupling various components, allowing a variety of toolchains and tools, modification to various parts of the system without affecting other parts, realtime adjustment of aspects such as routing through a variety of methods and methods to record aspects of the running system back into the higher level xml to store under version control, run elsewhere and improve upon.

An automatically generated diagram in the `generated` directory gives a sense of the generated permutations:
```
$ display permutations.svg
```

From the diagram(or grepping) we can see that there are gui versions (using kivy) for:

'pages' forwards
```
button-gui-kv-10.py
```

'pages' backwards
```
button-gui-kv-10.py
```

and then to slurp from the 'pages', which will try to slurp from any available/found device:

```
ma-throw slurp
```

which should return something like:
```
['glworb:42a6a43c-694d-461a-9a9c-1134bb64180b', 'glworb:5041b457-c27c-46af-8505-b34c142c9265'][]
```

The "glworb:"+uuid is a general datastructure stored as a redis hash. Mostly unexceptional, except that values may be keys to other redis keys.

# Using some tools

let's take a look using some ma-cli cli's:

look using the underlying db (redis):
```
ma-cli redis
> hgetall glworb:42a6a43c-694d-461a-9a9c-1134bb64180b
 1) "uuid"
 2) "42a6a43c-694d-461a-9a9c-1134bb64180b"
 3) "source_uid"
 4) "capture1"
 5) "method"
 6) "slurp_primitive_generic"
 7) "binary_key"
 8) "binary:058910b6-b0d5-41e9-ad4d-4cdbfd467443"
 9) "created"
10) "2018-03-23 08:31:43.205331"
```

look using visual interactions:
```
ma-cli image
> use glworb:42a6a43c-694d-461a-9a9c-1134bb64180b
> view
```
at the same prompt, draw a nonpermanent rectangle(x, y, w,h) on the image(useful for finding ocr regions):

```
>rectangle 40 40 200 200
>view
```

look another way using `ma-dm`:

```
ma-dm glworb:42a6a43c-694d-461a-9a9c-1134bb64180b --see-all
```

So far the process, simple or sophisticated is:
1. open 'advance' button permutation
2. run slurp
3. press 'advance' button to increment 'pages'
4. repeat

The process could be moved to hardware buttons, but for now a faster way using `ma-deck` to streamline the process before getting into the guis. We can create a yaml file that maps buttons to actions: 

tutorial_faster.yaml:
```
calls:
    slurp: ma-throw slurp
    pages_increment: ma-throw set BYTEBOOOK marker:capture* 2 + --service zerorpc-slurp-primitive-generic
    pages_decrement: ma-throw set BYTEBOOOK marker:capture* 2 - --service zerorpc-slurp-primitive-generic

bindings:
    SPACE: slurp
    LEFT: pages_decrement
    RIGHT: pages_increment
```

and run with:

```
ma-deck --yaml tutorial_faster.yaml
```
try pressing left, right and space!

