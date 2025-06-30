# Dumping packets to pcap format

Modifications needed to use the packet dumping in Artery:

* Modify the router in /artery/extern/vanetza/geonet/router.cpp and hpp with dumping funcionality
* Add parameter bool dumpPcap = default(false); in /artery/src/artery/networking/router.ned
* Add line if(par("dumpPcap"))  mRouter->set_dump(true); below the line mRouter->set_address(generateAddress(init_mac)); in Router.cc
* Enable dump for specific node in omnetpp.INI file such as: \*.rsu[0].vanetza[\*].router.dumpPcap = true

Note: mRouter->set_address() is called twice in Artery router, so two pcap files would be created. Therefore dumping is enabled after the first call of the function.