# Machinic

Rough outline and general philosophies:
* Encourage rapid and easy modification and prototyping
* Reduce friction wherever possible
* Loose, layered, moderately resilient, semi-structured, ecosystem
* Provide tools to feed modifications back into higher level specifications
* Generate code from high level specifications
* Provide high, mid- and low level tooling (especially to work with above points)
* Use DSLs as exploratory interfaces and provide as cli tools and packages
* Modular user interfaces that take advantage of shape and color, and use the same core tools as clis or other interfaces

## Installation

Install (and setup) everything:

This meta script should download other machinic repos and then: install, configure and run everything.

Useful on a fresh install of fedora, debian or raspbian.

Installation and configuration should be idempotent (for details of process see `environment.xml`).

```
git clone https://github.com/galencm/machinic-meta-install
cd machinic-meta-install
./install.sh
```

## Tutorial

This tutorial should serve as an introduction to the machinic ecosystem usage, concepts and tools through the scanning of a simulated book (a boook) using a process that should be seamlessly applicable to irl books.

Machinic-image has a source named 'BYTEBOOOK' to use for testing. It generates a _boook_, a set of images with a booklike structure: covers, table of contents, introduction with roman numerals, numbered pages with numbering appearing in corners and centers for new chapters. The source also stores marker positions of devices that can be incremented or decremented to simulate turning pages. This structure hopefully allows testing the entire system from slurping via hardware or software to defining the dss (using dss-ui) and then testing the queuing-ui and fold-ui for the mundane process of gathering and correctly arranging material.

References:

[machinic-image machine](https://github.com/galencm/machinic-image)

[boook](https://github.com/galencm/machinic-primitives-sources/blob/master/sourceprimitives/boook.py)

[indexable source](https://github.com/galencm/machinic-primitives-sources/blob/master/sourceprimitives/source_indexable.py)

### Getting started:

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

References:

[ma-cli](https://github.com/galencm/ma-cli)

### Simple and sophisticated processes

The processes involving the byteboook revolve around **slurping** material and _incrementing/decrementing_ the indexable positions. 

* Slurping is sort of a catchall term for pulling some material into the machinic ecosystem.
* Incrementing/decrementing will serve as the virtual equivalent of turning pages in an irl book.

These basic actions will be streamlined by creating buttons (and associated routes) to trigger slurping, and increment / decrementing. 

* Graphical interfaces (guis) might be used to visualize or inspect aspects of the process.
* Commandline interfaces / tools (clis) might be used to troubleshoot or modify the processes. 

A goal of the machinic ecosystem is that any arrangement of these processes should translate seamlessly into other configurations. So for example, the slurped byteboook sources become cameras above a book and the buttons to trigger slurping could be either hardware buttons or gui buttons.

Another goal of the machinic ecosystem is to allow flexible specification, generation, modification and enspecification(live modifications written back into the specification) to allow a variety of processes.

For this tutorial some approaches might look like:

* A simple process:
    1. Simulate turning page(s)
    2. Slurp
    3. repeat 1 & 2 until some threshold
    4. check slurped material
* A more sophisticated process:
    1. Simulate turning page(s)
    2. slurp some representative pages from various positions in the structure
    3. define domain specific structure using dss-ui
    4. use queue-ui and fold-ui to observe progress
    5. repeat 1 & 2 while doing 4

#### Starting simple:

Start in the generated directory of machinic-image:

```
cd machinic-image/generated
ls
```

`permutations.svg`, shows generated permutations and their relationships:
```
$ display permutations.svg
```

The generic sources have to be started by script to be discoverable:
```
python3 slurp-term-cli-6.py run generic
python3 slurp-term-cli-7.py run generic
```
These will slurp the left and right pages of the boook.

We needa way to "turn" the pages. From the diagram(or grepping) we can see that there are button guis for:

increment "pages" forwards
```
button-gui-kv-10.py
```

decrement "pages" backwards
```
button-gui-kv-11.py
```

run the gui buttons:
```
python3 button-gui-kv-10.py &
python3 button-gui-kv-11.py &

```

and then from the commandline slurp the "pages":

```
ma-throw slurp
```

which should return something like:
```
['glworb:42a6a43c-694d-461a-9a9c-1134bb64180b', 'glworb:5041b457-c27c-46af-8505-b34c142c9265'][]
```

**Glworbs** are redis hashes using naming scheme "glworb:" + a uuid. Glworb key values may be other redis keys, such as image bytes stored in keys using "binary:" naming. 

#### Getting a little more sophisticated:

So far the process, simple or sophisticated is:
1. open 'advance' button permutation
2. run slurp
3. press 'advance' button to increment 'pages'
4. repeat

The process could be moved to hardware buttons, but for now a faster way using `ma-deck` to streamline the process before getting into the guis. We can create a yaml file that maps buttons to actions: 

booklike_deck.yaml:
```
calls:
    slurp: ma-throw slurp
    slurp_with_preview: ma-throw slurp  | tr -d ",[]'"  | tr " " "\n" | ma-dm
    pages_increment: ma-throw set BYTEBOOOK marker:capture* 2 + --service zerorpc-slurp-primitive-generic
    pages_decrement: ma-throw set BYTEBOOOK marker:capture* 2 - --service zerorpc-slurp-primitive-generic

bindings:
    SPACE: slurp
    DOWN: slurp
    UP: slurp_with_preview
    LEFT: pages_decrement
    RIGHT: pages_increment
```

and run with:

```
ma-deck --yaml booklike_deck.yaml
```
try pressing left, right and space!

#### Tutorial notes

##### On `./generated`
_Where do `./generated/` files come from?_

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

##### On using some tools

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

