Introduction:

Machinic-image has a 'boook' source named 'BYTEBOOOK' to use for testing. It generates a set of images with a booklike structure: a cover, table of contents, introduction with roman numerals, numbered pages with numbering appearing in corners and centers for new chapters. The source also stores marker positions of devices that can be incremented or decremented to simulate turning pages. This structure hopefully allows testing the entire system from slurping via hardware or software to defining the dss (using dss-ui) and then testing the queuing-ui and fold-ui for the mundane process of gathering and correctly arranging material. 

This tutorial should serve as an introduction to the machinic ecosystem and various concepts and provide an approach that can be seamlessly used with realworld booklike structures...

Currently the installation of the machinic ecosystem(machinic-core, machinic-image, various ma-* packages) is not covered. They are assumed to be installed, this should be covered in another tutorial...

Rough ecosystem outline and general philosophies:
* Encourage rapid and easy modification and prototyping
* Reduce friction wherever possible
* Loose, layered, moderately resilient, semi-structured, ecosystem
* Provide tools to feed modifications back into higher level specifications
* Generate code from high level specifications
* Provide high, mid- and low level tooling (especially to work with above points)
* Use DSLs as exploratory interfaces and provide as cli tools and packages
* Modular user interfaces that take advantage of shape and color, and use the same core tools as clis or other interfaces

Basics:

_Let's see if BYTEBOOOK is running..._

Check if byteboook service is running using `ma-cli`. Ma-cli provides a variety of commandline interfaces some novel and some wrappers that supply host and port information. The `nomad` command mostly wraps the hashicorp `nomad` command with some additional commands such as `restart` and `metalogs`. 

```
ma-cli nomad
nomad:xxx.xxx.x.x:xxxx>status
```
should include:
```
ID                               Type     Priority  Status   Submit Date
byteboook                        service  50        running  03/09/18 13:17:50 UTC
```


Process:

A simple process for this tutorial might look like:
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

This tutorial will cover the second, longer process. 

```
cd machinic-image/generated
```

_Where do these things come from?_

relevant xml in `machinic-image/machine.xml`:
book and peripherals:
```
    <peripheral type = "button" alternative_press = " " description = "increment pages, simulate turning pages towards end">
        <output type = "integer" value = "+= 2" destination = "/set/BYTEBOOOK/marker:capture1" />
        <output type="integer" value="+= 2" destination = "/set/BYTEBOOOK/marker:capture2" />

    </peripheral>

    <peripheral type = "button" alternative_press = " " description = "decrement pages, simulate turning pages towards beginning">
        <output type = "integer" value = "-= 2" destination = "/set/BYTEBOOOK/marker:capture1" />
        <output type="integer" value="-= 2" destination = "/set/BYTEBOOOK/marker:capture2" />
    </peripheral>
    <!-- sources -->

    <source type="primitive_bytes_indexable" source="boook" location="BYTEBOOOK" wireup = "true">
        <set peripheral="capture1" symbol="marker" value="-1" overwrite="true" />
        <set peripheral="capture2" symbol="marker" value="0" overwrite="true" />
    </source>
```

viewers
```
    <peripheral type = "viewer">
        <input filter = "capture1" />
    </peripheral>

    <peripheral type = "viewer">
        <input filter = "capture2" />
    </peripheral> 
```

svg diagram of permutations code generation:
```
$ display permutations.svg
```

'pages' forwards
```
button-gui-kv-10.py
```

'pages' backwards
```
button-gui-kv-10.py
```

slurp
```
ma-throw slurp
```
should return something like:
```
['glworb:42a6a43c-694d-461a-9a9c-1134bb64180b', 'glworb:5041b457-c27c-46af-8505-b34c142c9265'][]
```

let's take a look using some ma-cli cli's:

the underlying db:
```
ma-cli redis
> hgetall glworb:42a6a43c-694d-461a-9a9c-1134bb64180b
```

visual interactions:
```
ma-cli image
> use glworb:42a6a43c-694d-461a-9a9c-1134bb64180b
> view
```

or

```
ma-dm glworb:42a6a43c-694d-461a-9a9c-1134bb64180b --see-all
```

Slow way:
open button1 forwards
run slurp, press button...


Faster way:
ma-deck

1. create yaml
2. run
set -> slurp_primitive_generic.py

!!!TODO get set to work correctly with sources...
currently it works on discovered devices...