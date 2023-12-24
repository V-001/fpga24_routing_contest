# Class NxRoutingGraph

This should be the graph that represents the physical netlist after being parsed and built. Note that the implementation provided is in python and used the networkx package, this implementation is simply to show that we are able to extract information from the a file that used the FPGAIF schema. Seens to very often reference the device resources schema specifically. 

The routing graph itself shoud on be derived from the device resources schema

### class CustomEdgeAttribute
I DON'T UNDERSTAND PYTHON DATA STRUCTURES :SOB:
### class CustomNodeAttribute
I DON'T UNDERSTAND PYTHON DATA STRUCTURES :SOB:
### def build(self, filename)

This functions inherits itself as we are building the graph object. As far as I understand there should only be one graph object per netlist, but **typically** we should only be building a singular netlist (according to sources (God?) it should be the physical netlist). It also gets fed the FPGAIF file (allegedly (source: God?))

We do ``item = self.item`` a lot in this function. From what I can gather this is a pythonic mathod of creating a pointer to the item within the object. This way we are being better with memory access and privilege within our function. (source: me thinks )

    Note: Why do we have "types" for sites and tiles if we are allegedly taking in the physical netlist? For now assume that we are taking in the logical netlist or device resouces? After careful examination of the build funcation I think this class/object solely used the device resources schema

    More Notes: 
    pip == programable interconnect point, basicaly allows us to have a connection between two wires (uni/bi directional is possible)

    Seems to be vary improvable in terms of runtime through the use of getters. Might be easier to build in a different language though? Also aim to make this more readable. 

1) We create 4 variables within the object
    - self.tileType2SiteTypePinName2wire is a set(?) that contains the siteTypes for each tile and the pinName (pin) for each site. This way we are able to locate the site on the physical hardware and wire it
    - self.self2tileAndTypes is a set(?) that I don't really know what it does
    - self.tile2wire2node is a set(?) that maps the tile (highest level) to the wire(s?) that it should be connected to?
    - self.pipData is a list of wires?

2) Open the file

3) Import the DeviceResourcesschema so we are able to determine what hardware we are working with
    - Do we unload all the data in a round robin fashion or dump all of one type and then move to the next? 

4) Read the device resources
    - Set your coordinates with self.MIN_X/Y and self.MAX_X/Y
    - compile the name of each tile and add it to a list of tiles
        - this is local to the build function but does not get stored to the graph object yet
5) This chunk of code
    ```wires = device.wires
    add_node = self.add_node
    tile2wire2nodeSetdefault = self.tile2wire2node.setdefault
    ```
    - Here's what I think this is doing. We are getting a *list* of wires from the device resources schema. We are then blocking about memory to create an empty node within our object and preping to map our tiles to nodes using the list of wires we just recieved from the schema. Me thinks. 

6) ``for nodeIdx,node in enumerate(device.nodes):``
    - nodeIdx is the loop idx starting from 0
    - node should be the value from device.nodes? (someone fact check me pls)
    - This loop is filling out the ``tile2wire2nodeSetdefault`` variable 
        - This is essentially building the graph nodes from our device resouces? 
            - Why??? Shouldn't this be happening in the netlist or is the netlist using the graph we are making now. I thought we were fitting the device to the netlist, not the netlist to the device...

7) set up pointers(?) to the items within our object so that we can insert edges into our graph

8) loop through all the tiles ``for tile in tiles:``
    - in this loop we first check if there are nodes within the tile, if not we continue to the next tile
        ```
        wire2node = tile2wire2nodeGet(tile.name)
            if wire2node is None:
                # No nodes in this tile
                continue 
        ```
    - acquire the tileType and the wire2node
    - Now loop through all the pips for the tileType (the hell is a pip? is it a type of edge? )
        - We ignore 'conventional' pips
        - if there is no wire a pip is unable to exist (I think this means that edges are equivalent to edges in our graph)
        - if we don't see the wire in our list(?) of wires then add it through the default method ``pipData2indexSetdefault((wire0Name,wire1Name,forward), len(pipData))``
            - This seems to always happen though? So I guess we haven't added any wires to our wire list(?) yet? 
        - if we adding a new wire then append the list with the names and forward = true
        - if the pip is bidirectional then set forward = false and add another edge connected to the same nodes in reverse so we can imitate bidirectionality
    - I think that's it? 

9) Build mapping from the site to the pin idx to the pin name
    - why? 
    - Is this in the custom node attributes? 

10) loop through the tileTypeList (tileTypeIdx is the loop idx starting from 0, tileType is the data from device.tileTypeList)
    - This loop is nested to an ungodly amount, runs in O(T * S * W) time :skull:
    - Essentialy mapping our tiles to the siteType pin (which should just be the site pin?) and the wire the site is connected to... I think

11) go through all the tiles again
    - Is there a site in this tile? if not then lets move to the next tile
        - No point in running the other stuff if no sites within our tile
    - lets loop through the sites within our tile now
        - take in the site name go the item in the list/set self.site2tileAndTypes for it. Assign the following value to it... (tile.name,tile.type,site.type)

12) This chunk of code 
    ```
    # Keep only site pin wires in self.tile2wire2node
        siteTypePinName2wire = self.tileType2SiteTypePinName2wire[tile.type]
        sitePinWires = set(siteTypePinName2wire.values())
        wire2node = {k:v for k,v in tile2wire2node[tile.name].items() if k in sitePinWires}
        tile2wire2node[tile.name] = wire2node 
    ```
    - I don't really feel like understanding this even though I really should. It look important

13) end of def build

### def getNodesFromSitePin(self, siteName, pinName) 
Never called for some reason. Self explanatory

### def getPIP(self, u, v)
Called in NxRouter::write only for some reason. Self explantory in terms of function, however not clear as to what ``u`` and ``v`` are? 

# class NxRouter
Should only be using the device resources and physical netlist schemas. This class(its more like a factory as opposed to an object if anything but anyways) should create a routing graph from the device resources by creating a new NxRoutingGraph object. This (I think) will be manipulated by the class to create a valid physical netlist from the current physical netlist. 

### def create(deviceResourcesFilename, physNetlistFilename)
This function creates itself using the device resouces provided by the given file? This actually makes no fucking sense. 

Then we parse the physical netlist and yield to the object itself? The fuck????? 

### def __init_(self, deviceResourcesFilename)
- Create an empty routing graph
- Build the routing grpah

### def parse(self, netlist)
god help us

    Notes: 
    stub branches are unrouted site pins that we can use for routing. They are NOT leaf nodes

1) assign the recieved netlist to the object

2) Create an empty set(?) self.net2pin2node to map a source pin to a node to a sink node? 
    ```
     source pin node -> node -> sink node
    ```

3) we are going to loop through each net in our physical netlist schema
    - the rest of this function is within this loop

4) check if the net type is a signal and we have stub nodes 
    - if true then continue on to 5.
    - if false then the net is fully routed or it is a gnd/vcc net and remove it from the routing graph (unless unique circumstance TODO: FIND THE UNIQUE CIRCUMSTANCES) 

5) Check is we have sink pins, if we don't then go to the next net

6) loop through all our source pins
    - in this loop we want to map all our source pins to the correct node
        - This should be relatively easy since we've already built the routing graph and each site pin is related to a node, so we are just collecting the nodes that are related to a source pin

7) loop through all our sink pins
    - check if it exists
    - check if the net has no sources, if yes then it's unroutable and should be removed from the routing graph. Like why would you have sinks if you don't have sources... :skull:
    - otherwise add it to the list of sink pin nodes in the same way we did for the source pins
        - remove all outgoing edges from the sink pin node. Since the sink pin is in essence a termination point, we don't want anything to be coming out of it.
            - this might cause vivado issues? I don't know why bc I am not a vivado connisewer or whatever 

8) ```
    self.net2pin2node[net.name] = (sourcePin2node,sinkNodes)
    ```

9) parse finished
### def route(self)
This is the actual routing portion of the poc router. Note that this is a god damn unga bunga ass implementation and probably is NOT the best possible implementation that we could create. However, it is a good baseline for what we could have. Might be worth our time to see if we can adapt parts of the RapidWright router to our Rust router. RustWright if you will. 

This implementation also produce sa partially valid solution rather than a valid solution. Some nets have too many connections.

    Notes: 
    I don't know what hiddenEdges are. I think these are just edges that connect the same two nodes making it an invalid route? We will have to deal with these a lot in our implementation I think... seems almost unavoidable

    This implemenation uses given functions within the networkx package... maybe this is why it produces a partially valid solution? perchance?

    Why are we allowing a node to be driven in two different nets? Or are we not allowing that here an I'm actually insane... very possible meow :3

1) loop through each item in self.net2pin2node.items() 
    - this whole function occurs within this loop
    - netName is the name of the net
    - (sourcePin2Node, sinkNodes) 
        - sourcePin2Node is the mapping from the source node to the node with our BEL/Cell 
        - sinkNodes is the list of sinkNodes within our net

2) loop through each sink node
    - set the path = node as we need to make the path 

3) loop through each source node

4) Use nx.shortest_path on the graph between the source node and the sink node
    - if there is no path then move to the next source node

5) if we are unable to route the sink pin then move to the next sink node

6) go through the path between the source and sink nodes in the graph (at least that's what I think is happening)
    - set each node in the path to the default (or in our implemenation something funny like unga bunga or meow :3)
    - if we detect a multisink go insane? 
        - we don't want a node to be driven by two different things
        - so in our implementation we have to prevent this but here they just kind of ignore it.... or I might be wrong and misunderstanding 
7) increment num of pins routed

8) After routing the entire net restore the hiddenEdges we banished to the shadow realm and clear the list containing them

9) we did it!

### def write(self, filename)
converts physical netlist graph thing into physical netlist schema for FPGAIF lmao

I really don't feel like going through this function and figuring out exatly how it works since we won't be using python but rust for this. so lmao

### def extractSitePins(self, branches)
Get all the site pins from the entire device? This seems very useful

### def getStringIndex(self, string)
Self explanatory 
