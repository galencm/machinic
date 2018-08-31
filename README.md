# Machinic

## Notes on some machinic packages

* machinic
    * machinic light (abstract term)
        * emerged out of useful and difficult aspects of machinic heavy
        * while heavy is more abstract, light's initial guis are oriented around the capturing and sequencing of images
        * use guis to illustrate how pieces of the ecosystem fit together
        * use guis to explore visualization and interactions difficult on commandline (see `fold-ui`)
        * all aspects should be easily (and observably) runnable on a single box
        * uses the host:port combination of the db (redis) as a namespace
        * guis to tie together functionality for common tasks (such as `dzz-ui`)
        * beneath, beside and beyond guis: dsls, code generation and cli tools available to vastly extend possibilities as needed
    * machinic heavy (abstract term)
        * original development of machinic (before heavy/light distinction)
        * machines described as xml models and then instantiated using code generation as a mix of requirements, services, programs and other generated code
        * modification and regeneration of machines on the fly
        * see `machinic-core` and `machinic-image` to get a sense of structure
        * code provided as services using zerorpc
        * service discovery with consul
        * service scheduling with nomad
        * variety of commandline tooling to deal with layers of services, machines, and so on
    * fold-lattice-ui
        * an item (or glworb) is a redis hash key used across machinic tools 
        * visualize and interact with items
        * allow a flexible system of views per item (edit, script, image) that can be tabbed through
        * specify expected amount of items and highlight missing
        * specify order of items and highlight discontinuities
        * color code items based on presence or absence of values
        * modify item values using keyling dsl
        * use keyling and `machinic-keli` to pass items to shell commands
        * generate testing material using `fold-ui-fairytale`
    * dzz-ui
        * create classification or item data that can be used by `fold-ui`
        * select ocr regions on an image blob
        * create scripts to handle cropping, ocr-ing and classifying ocr result
    * machinic-tangle
        * tangle-ui
            * ui overview of db and broker messaging
            * ui overview of scanning and AP connections
            * ui editing of routes in pathling
            * run broker if needed
            * run access point to provide access to broker
            * handle routing between broker and db using pathling
            * scan for unconfigured devices
            * associate with unconfigured devices and provide configuration
        * tangle-things
            * generate things and code for things
            * things can have a variety of forms such as a gui button or an esp82666 with a physical clicky button
            * modify and regenerate
    * enn-ui
        * enn-dev
            * gui to discover and configure devices (such as chdk cameras)
            * create configurations conditional on environment values that can be evaluated before device captures (see `neo-slurpif` in machinic-keli)
        * enn-env
            * gui to see and modify environment values
            * environment values are a simple hash key
    * ma-cli
        * commandline tools originally designed for machinic heavy and associated infrastructure
        * `ma-cli` provides access a variety of interactive clis
        * `ma-dump` to store state of system in xml and `ma-load` to deploy routes, scripts, ...
    * ma
        * code generation tools originally designed for machinic heavy
        * generate install/system check scripts for a variety of platforms from xml (see `machinic-meta-install`)
        * generate machines and their associated scripts from an xml model
