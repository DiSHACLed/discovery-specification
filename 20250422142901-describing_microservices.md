
# Table of Contents

1.  [Abstract](#org47728b9)
2.  [Considerations](#org4732e53)
    1.  [Use cases](#orgd156bf1)
        1.  [Microservice discovery](#org556972c)
        2.  [Connecting microservices](#org710c615)
        3.  [Generic tronsformations](#org9fd5e45)
        4.  [Validating authorization rules](#org8d38111)
    2.  [Interop with datasets](#org3da6960)
    3.  [Similarity with typing systems](#org9ee40c0)
    4.  [Simplicity and automation](#org8d604be)
3.  [Which information to capture](#org616fb4a)
4.  [DCAT and SHACL as a basis for describing data processes](#org9abe7e5)
5.  [Capturing the information](#org3d4ecf0)
    1.  [Image service](#org6add9c7)
    2.  [Processing catalog](#org1b5df69)
6.  [Information storage](#orgba346dc)
    1.  [Adding semantic metadata to the source-code](#org33e16f8)
    2.  [Retrieving information from a Docker Image](#orgb8ff9ec)
        1.  [semantic.works](#org29929f1)
        2.  [unix filesystem](#org42ab8fc)
7.  [Examples](#org2388a91)
    1.  [Image service (described above)](#org6e9a824)
8.  [Conclusion](#org7298394)


<a id="org47728b9"></a>

# Abstract

Microservices, like other software applications, treat some form of data.  Some consume data, some produce data, many do both.  Multiple processes can exist within a single service.  Here we describe the ways in which microservices can be annotated in order to describe their inputs an outputs.


<a id="org4732e53"></a>

# Considerations


<a id="orgd156bf1"></a>

## Use cases


<a id="org556972c"></a>

### Microservice discovery

By describing microservices a full image of the information in the system can be constructed, thus allowing for the discovery of relevant microservices.  This helps in systems such as semantic.works.  Eg: if emails are written into an outbox, various email sending services can be discovered.


<a id="org710c615"></a>

### Connecting microservices

In solutions which wire microservices to each other, pipelines can be created by connecting compatible processes.  This helps in building larger pipelines from LDES clients.  Eg: an LDES feed containing individual individual cyclists passing through a street could be sent through a service which emits a warning when there is likely a traffic jam.


<a id="org9fd5e45"></a>

### Generic tronsformations

Some services are generic in nature.  They take in some configuration or datastream and emit another datastream.  Referring to aspects of the input stream may be valuable for describing the output stream, similar to generic typing.  Eg: the previously mentioned LDES feed containing individual cyclists passing through a street could be sent through a generic summarization service which creates a daily summary of the amount of cyclists linked to each individual measurement.


<a id="org8d38111"></a>

### Validating authorization rules

Knowing the inputs and outputs of processes, validation systems can verify this information may be written to shared databases.  This allows users to verify up front whether the authorization rules are compatible with the target processes.  Eg: if a process which emits summarized gender information based on individual gender datap-points.  It will fail to operate without these data-points and this can be determined based on static analysis.


<a id="org3da6960"></a>

## Interop with datasets

A process may consume and generate data.  Whether this data comes from the instantiation of a microservice or from an existing dataset makes little difference from a data processing perspective.  The solution should be applicable similarly for datasets as well as for data processes.


<a id="org9ee40c0"></a>

## Similarity with typing systems

The description of microservices should allow for a human understanding of the microservice's function as well as a machine-based calculation of the resulting data feeds.  We can draw from a rich research around typing systems and use that as a basis for an easy understanding.


<a id="org8d604be"></a>

## Simplicity and automation

Documentation isn't always considered to be the most valuable task.  As such, the barrier to entry should be kept low.  It should be possible to describe complex cases and trivial to describe simple cases.


<a id="org616fb4a"></a>

# Which information to capture

We need to capture information for machine-based connection of services' input and output streams, as well as a description for humans to determine the meaning of the processing.  As a microservice we may describe the software component, or an instantiation.  Further, a microservice may have multiple input and output streams.


<a id="org9abe7e5"></a>

# DCAT and SHACL as a basis for describing data processes

DCAT is well suited for describing datasets.  A key target is interoperability between services and datasets, hence we aim to use the DCAT vocabulary.  DCAT can be used to describe:

-   microservices
-   microservice data processes
-   microservice instantiations
-   endpoints of microservice data processes

DCAT becomes the index in which processes are described.

SHACL will be used to describe the input and output shapes of each microservice process.  The union of these input shapes is the input shape of the full microservice, the union of the output shapes becomes the output of the full microservice.  The SHACL shape can serve as a typing system for connecting processes.

In order to describe the equivalent of generic types, the URIs for the input shapes can be reused in output shapes.  A reasoning service can then derive that the additional constraints placed on the input shape also apply to the output shape.


<a id="org3d4ecf0"></a>

# Capturing the information

Ignoring base models, we want to indicate the existence of a microservice with basic operations and corresponding data streams.


<a id="org6add9c7"></a>

## Image service

As an example we may consider the image service.  This service converts image files between sizes.

    @prefix dct: <http://purl.org/dc/terms/>.
    @prefix ext: <http://mu.semte.ch/vocabluries/ext/>.
    @prefix mu: <http://mu.semte.ch/vocabularies/core/>.
    @prefix nfo: <http://www.semanticdesktop.org/ontologies/2007/03/22/nfo#>.
    @prefix nie: <http://www.semanticdesktop.org/ontologies/2007/01/19/nie#>.
    @prefix sh: <http://www.w3.org/ns/shacl#>.
    @prefix xsd:  <http://www.w3.org/2001/XMLSchema#>.
    
    <https://semantic.works/services/image-service>
      a mu:Microservice;
      dct:title """Image service""";
      dct:description """Providesscaled down versions of imageson the fly and caches the results in the triplestore.""";
      ext:hasProcess
        <https://semantic.works/services/image-service/process/resize> ,
        <https://semantic.works/services/image-service/process/retrieve>;
      ext:input [ sh:or (:unconvertedInput :convertedOutput) ];
      ext:output :convertedOutput.
    
    <https://semantic.works/services/image-service/process/resize>
      a ext:Process;
      dct:title "Resize image";
      dct:description """Resizes an image and returns it to the user.""";
      ext:input :unconvertedInput;
      ext:output :convertedOutput.
    
    <https://semantic.works/services/image-service/process/retrieve>
      a ext:Process; 
      dct:title "Retrieve resized image";
      dct:description """Renders a previously resized image""";
      ext:input :convertedOutput.
    
    :unconvertedInput
      a sh:NodeShape; # TODO: sh:NodeShape applies to _all_ nfo:FileDataObject, yet we only care about some of these matching.  Whether _everything_ is expected to match or _some_ are expected to match is a desired extension.  We may want to ignore what does not, or we may crash if it does not match.
      sh:targetClass nfo:FileDataObject ;
      sh:property [
        sh:path mu:uuid ;
        sh:minCount 1 ;
        sh:dataType xsd:string
      ] ;
      sh:property [
        sh:path dct:format ;
        sh:minCount 1 ;
        sh:dataType xsd:string ;
        sh:pattern "^image/"
      ] ;
      sh:property [
        sh:path [ sh:inversePath nie:dataSource ] ;
        sh:minCount 1
      ] ;
      sh:closed false .
    
    :convertedOutput
      a sh:NodeShape ;
      sh:targetSubjectsOf ext:hasDerivedImage ; # Original file
      sh:property [
        sh:path ext:hasDerivedImage ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:node [
          a sh:NodeShape ;
          sh:targetClass nfo:FileDataObject ;
          sh:property [
            sh:path ext:imageWidth ;
            sh:minCount 0 ;
            sh:maxCount 1 ;
            sh:dataType xsd:string
          ] , [
            sh:path ext:imageHeight ;
            sh:minCount 0 ;
            sh:maxCount 1 ;
            sh:dataType xsd:string
          ] , [
            sh:path nie:dataSource ;
            sh:minCount 0 ;
            sh:maxCount 1 ;
            sh:nodeKind sh:IRI ;
          ]
        ]
      ] ;
      sh:closed false.

With example (incomplete) payload:

    @prefix dct: <http://purl.org/dc/terms/>.
    @prefix ext: <http://mu.semte.ch/vocabluries/ext/>.
    @prefix mu: <http://mu.semte.ch/vocabularies/core/>.
    @prefix nfo: <http://www.semanticdesktop.org/ontologies/2007/03/22/nfo#>.
    @prefix nie: <http://www.semanticdesktop.org/ontologies/2007/01/19/nie#>.
    @prefix sh: <http://www.w3.org/ns/shacl#>.
    @prefix xsd:  <http://www.w3.org/2001/XMLSchema#>.
    
    :foo ext:hasDerivedImage :bar


<a id="org1b5df69"></a>

## Processing catalog

Each of the services and each of the processing steps can be retrieved through one or more DCAT catalogs.  Such a Catalog allows for the discovery of various processes and thus needs to have access to the SHACL shapes.

    :semantic-works-services
      a dcat:Catalog;
      dct:title "Public semantic.works microservices and processes.";
      dct:description "Contains a distribution for all DiSHACLed annotated microservices and processes.";
      dcat:dataset
        <https://semantic.works/services/image-service>,
        <https://semantic.works/services/image-service/process/resize> ,
        <https://semantic.works/services/image-service/process/retrieve>.
    
      <https://semantic.works/services/image-service>
        a dcat:Distribution, mu:Microservice;
        dcat:distribution <https://github.com/madnificent/mu-image-service>.
    
      <https://github.com/madnificent/mu-image-service>
        a dcat:Distribution;
        dct:title """mu-image-service git forge";
        dcat:accessUrl <https://github.com/madnificent/mu-image-service>.
    
    # TO BECONTINUED


<a id="orgba346dc"></a>

# Information storage

The information describing a microservice should be stored together with the microservice itself.  Ideally the information can be stored together with the source-code and with the built image.

The following is based on the way services are managed within semantic.works.  Although some of this will not map to other systems, most of it should.


<a id="org33e16f8"></a>

## Adding semantic metadata to the source-code

semantic.works has a simple structure for constructing microservices.  A readme is generally stored in a README.md file.  Akin to the web we suggest creating an `doc/meta.ttl` or a top-level `meta.ttl` in which the sources can be described and from which references can be made to other files if necessary.


<a id="orgb8ff9ec"></a>

## Retrieving information from a Docker Image

A running container could have the same information embedded.  Different frameworks will likely provide this information in different places.


<a id="org29929f1"></a>

### semantic.works

semantic.works places the application in `/app` for a consistent behaviour.  Scripts are stored in `/app/scripts` making `/app/doc/meta.ttl` or `/app/meta.ttl` logical entryoints for this descriptive metadata.


<a id="org42ab8fc"></a>

### unix filesystem

Frameworks which don't hold such a convention may choose to follow the well thought out Linux filesystem hierarchy.  In current times a subfolder of `/usr/share/doc` would be a good place to store the `meta.ttl`.   It is not readily obvious if a standard subfolder could be imagined and what that would be named.  Without a fixed determination of said folder, discovery is hard.


<a id="org2388a91"></a>

# Examples


<a id="org6e9a824"></a>

## Image service (described above)

The image service is a microservice which consumes image files and which can produce scaled down versions of these files.


<a id="org7298394"></a>

# Conclusion

Microservices can be described using DCAT and extensions to DCAT.  A structure of this kind allows to 

