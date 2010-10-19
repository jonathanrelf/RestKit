Introduction
=========================

RestKit is a library for interacting with Restful web services in Objective C. It provides a set of primitives for interacting with web services wrapping GET, POST, PUT and DELETE HTTP verbs behind a clean, simple interface. RestKit also provides a system for modeling remote resources by mapping them from JSON payloads back into domain objects. Object mapping functions with normal NSObject derived classes with properties. There is also an object mapping implementation included that provides a Core Data backed store for persisting objects loaded from the web.

RestKit was first publicly introduced in April of 2010 in a blog post on the Two Toasters website:
[http://twotoasters.com/index.php/2010/04/06/introducing-restkit/](http://twotoasters.com/index.php/2010/04/06/introducing-restkit/)

To get started with installation, skip down the document below the Design & Dependencies section.

Design
-------------------------

RestKit is composed of 3 main pieces: **Network**, **Object Mapping**, and **Core Data**. Each layer provides a higher level of abstraction around the problem of accessing web services and representing the data returned as an object. The primary goal of RestKit is to allow the application programmer to think more in terms of their application's data model and less about the details of fetching, parsing, and representing resources. Functionally, each piece provides...

1. **Network** - The network layer provides a request/response abstraction on top of NSURLConnection. The main interface for the end developer is the *RKClient*, which provides an interface for sending GET, POST, PUT, and DELETE requests asynchronously. This wraps the construction and dispatch of *RKRequest* and *RKResponse* objects, that provide a nice interface for working with HTTP requests. Sending parameters with your request is as easy as providing an NSDictionary of key/value pairs. File uploading support from NSData and files is supported through the use of an *RKParams* object, which serializes into a multipart form representation suitable for submission to a remote web server for processing. SSL & HTTP AUTH is fully supported for requests. *RKResponse* objects provide access to the string of JSON parsed versions of the response body in one line of code. There are also a number of helpful method for inspecting the request and response such as isXHTML, isJSON, isRedirect, isOK, etc.
1. **Object Mapping** - The object mapping layer provides a simple API for turning remote JSON responses into local domain objects declaratively. Rather than working directly with *RKClient*, the developer works with *RKObjectManager*. *RKObjectManager* provides support for loading a remote resource path (see below for discussion) and calling back a delegate with object representations of the data loaded. Remote payloads are parsed to NSDictionary representation and are then mapped to local objects using Key-Value Coding. Any class implementing the *RKObjectMappable* protocol can be object mapped. For convenience, RestKit ships with a *RKObject* implementation that can serve as a turn-key superclass for modeling your resources. You need only inherit & implement *elementToPropertyMappings* to begin loading resources via *RKObjectManager*'s *loadObjectsAtResourcePath:objectClass:delegate:* method.
1. **Core Data** - The Core Data layer provides additional support on top of the object mapper for mapping from remote resources to persist local objects. This is useful for providing offline support, holding on to transient data, and speeding up user interfaces by avoiding expensive trips to the web server. The Core Data support requires that you initialize an instance of *RKManagedObjectStore* and assign it to the *RKObjectManager*. For each persistent class you wish to model, you need to inherit from *RKManagedObject* and implement the appropriate *RKObjectMappable* methods. See the Examples/ subdirectory for examples of how to get this running. The Core Data support also provides *RKObjectSeeder*, a tool for creating a local "seed database" to bootstrap an object model from local JSON files. This allows you to ship an app to the store that already has data pre-loaded and then synchronize with the cloud to keep your clients up to date.

### Base URL and Resource Paths

RestKit utilizes the concepts of the Base URL and resource paths throughout the library. Basically the base URL is a prefix URL that all requests will be sent to. This prevents you from spreading server name details across the code base and repeatedly constructing URL fragments. The *RKClient* and *RKObjectManager* are both initialized with a base URL initially. All other operations dispatched through these objects work of a resource path, which is basically just a URL path fragment that is appended to the base URL before constructing the request. This allows you to switch between development, staging, and production servers very easily and reduces redundancy.

Note that you can send *RKRequest* objects to arbitrary URL's by constructing them yourself.

Dependencies
-------------------------

RestKit provides JSON parser implementations using SBJSON & YAJL. The recommended parser is YAJL (as it is known to be faster), but you can use the SBJSON backend instead by adding a dependency on libRestKitJSONParserSBJSON linking against libRestKitJSONParserSBJSON.a instead of the libRestKitJSONParserYAJL.a

RestKit also links against RegexKitLite to provide regular expression matching (used by the dynamic router).

The sources for SBJSON, YAJL and RegexKitLite are included in the Vendor/ subdirectory. The headers are copied into the RestKit headers path at build time and can be imported into your project via:
    #import <RestKit/Support/JSON/SBJSON/JSON.h>
    #import <RestKit/Support/JSON/YAJL/YAJL.h>
    #import <RestKit/Support/RegexKitLite.h>

Currently bundled version of these dependencies are:

* **YAJLIOS** - 0.2.21
* **SBJSON** - 2.3.1
* **RegexKitLite** - 4.0
  
If you currently link against or include SBJSON or YAJL in your project, you can disable the RKJSONParser targets and compile the appropriate RKJSONParser implementation directly into your application.

XML parsing is not currently supported.

Additional parsing backend support is expected in future versions.

Documentation & Example Code
-------------------------

Documentation and example code is being added as quickly as possible. Please check the Docs/ and Examples/ subdirectories to see what's available. The [RestKit Google Group](http://groups.google.com/group/restkit) is an invaluable resource for getting help working with the library.

Installation
=========================

Quick Start (Git Submodule)
-------------------------

To add RestKit to your project (you're using git, right?):

1. Add the submodule: `git submodule add git://github.com/twotoasters/RestKit.git RestKit`
1. Open RestKit.xcodeproj and drag the RestKit project file into your XCode project.
1. Click on the entry for RestKit.xcodeproj in your project's **Groups & Files** section. In the right hand pane, find the entries for **libRestKitSupport.a** **libRestKitObjectMapping.a** **libRestKitNetwork.a** and **libRestKitJSONParserYAJL.a** and click the checkboxes on the far right underneath the silver target icon. This will link your project against RestKit. If you wish to use the Core Data support, click the checkbox next to **libRestKitCoreData.a** also (this requires that you add Core Data to the Linked Libraries section at the bottom of the screen as well).
1. Get Info on your target and you should be looking at the **General** tag. In the top **Direct Dependencies** section, click the plus button and add a direct dependency on the RestKit target.
1. On the bottom pane of the General tab, click the plus button and add **libicucore.dylib** to the Linked Libraries.
1. Switch to the 'Build' tab in your project inspector. Make sure that your **Configuration** pop-up menu reads **All Configurations** so that your changes will work for all build configurations. 
1. Find the **Header Search Paths** setting. Double click and add a new entry. When RestKit is compiled, it will copy all relevant headers to the appropriate location under the /Build directory within the RestKit checkout. You need to add a path to the /Build directory of RestKit, relative to your project file. For example, if you checked the submodule out to the 'Libraries' subdirectory of your project, your header path would be 'Libraries/RestKit/Build'.
1. Now find the **Other Linker Flags** setting. Double click it and add entries for -all_load and -ObjC.
1. You may now close out the inspector window.

Congratulations, you are now done adding RestKit into your project!

You now only need to add includes for the RestKit libraries at the appropriate places in your application. The relevant includes are:
    #import <RestKit/RestKit.h>
    // And if you are using Core Data...
    #import <RestKit/CoreData/CoreData.h>

Please see the Examples/ directory for details on utilizing the library.

Detailed Installation
-------------------------

TODO

Contributing
-------------------------

Forks, patches and other feedback are always welcome. 

A Google Group for development and usage of library is available at: [http://groups.google.com/group/restkit](http://groups.google.com/group/restkit)

### RestKit is brought to you by [Two Toasters](http://www.twotoasters.com/). ###
