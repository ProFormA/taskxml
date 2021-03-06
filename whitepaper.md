# An XML exchange format for (programming) tasks

**Version 1.0.1**

contributors listed in alphabetical order:


Humboldt-Universität zu Berlin: *Niels Pinkwart, Sven Strickroth*
Technische Universität Clausthal: *Oliver Müller*
Universität Duisburg-Essen: *Michael Striewe*
Hochschule Hannover: *Sebastian Becker, Oliver J. Bott, Robert Garmann, Peter Werner*
Universität Osnabrück: *Helmar Gust, Nadine Werner*
Ostfalia Hochschule für Angewandte Wissenschaften: *Stefan Bisitz, Stefan Dröschler, Nils Jensen, Uta Priss, Oliver Rod*

[TOC]

## Introduction

This document specifies syntax and semantics for a standardized “Task
Format” - an exchange format for programming exercises/tasks. It
formalises the specification of programming exercises, for example, the
description, required programming language and suggested tests for
evaluating the student submitted code so that exercises written for one
tool can be exported and imported into another tool. The main
e-assessment tools considered for this format are Praktomat, VIPS, JACK,
GATE or Moodle-Grapper WS (inspired by Web-CAT) most of which are
included in the eCULT project “ProFormA”. The latest version of this
document and the corresponding XML Schema can be found at this address:
https://github.com/ProFormA/taskxml/

## General Structure

The XML exchange format consists of two parts: The first section shows
the description/specification of a task, including supporting files; and
the second section demonstrates a specification of tests which are to be
included in the specification of a task under the \<tests\> tag. Each
task can have many tests.

## Format specification

The XML file can be exchanged as a standalone file. However, if it
refers to several files, it is recommended to combine the XML file and
all linked files into a ZIP-archive. The XML file itself should be
stored in the root of the ZIP under the name “task.xml” so that it can
be found by tools supporting this format. The XML file should be well
formed and encoded in UTF-8 (RFC 3629). ID’s are globally unique within
the XML document.

## Task section

### XML Specification

The general structure of the XML format is given as follows (this is
meant to provide an overview and does not represent a minimal document):
```xml
    <tns:task lang="[LANG code]" xmlns:tns="urn:proforma:task:v1.0.1">
        <tns:description></tns:description>
        <tns:proglang version=""></tns:proglang>
        <tns:submission-restrictions />

        <tns:files />

        <tns:external-resources />

        <tns:model-solutions />

        <tns:tests />

        <tns:grading-hints />

        <tns:meta-data />
    </tns:task>
```

The following code shows the XML Schema for the Task Format:

```xml
    <xs:element name="task">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="tns:description"/>
                <xs:element ref="tns:proglang"/>
                <xs:element ref="tns:submission-restrictions"/>
                <xs:element ref="tns:files"/>
                <xs:element ref="tns:external-resources" minOccurs="0"/>
                <xs:element ref="tns:model-solutions"/>
                <xs:element ref="tns:tests"/>
                <xs:element ref="tns:grading-hints" minOccurs="0"/>
                <xs:element ref="tns:meta-data"/>
            </xs:sequence>
            <xs:attribute name="lang" type="xs:string" use="required"/>
        </xs:complexType>
        <xs:keyref name="tests-filerefs-fileref" refer="tns:fileid">
             <xs:selector xpath="tns:tests/tns:test/tns:test-configuration/tns:filerefs/tns:fileref"/>
             <xs:field xpath="@refid"/>
        </xs:keyref>
        <xs:keyref name="tests-extresrefs-extresref" refer="tns:external-resourceid">
             <xs:selector xpath="tns:tests/tns:test/tns:test-configuration/tns:externalresourcerefs/tns:externalresourceref"/>
             <xs:field xpath="@refid"/>
        </xs:keyref>
        <xs:keyref name="modelsolutions-model-solution-filerefs-fileref" refer="tns:fileid">
             <xs:selector xpath="tns:model-solutions/tns:model-solution/tns:filerefs/tns:fileref"/>
             <xs:field xpath="@refid"/>
        </xs:keyref>
    </xs:element>
```
	
The document root element “task” holds the XML-namespace URI for the
current version number of the XML Task Format. The only currently valid
value is “urn:proforma:task:v1.0.1”.

### Task attributes

The task is identified by attribute "uuid", an automatic generated UUID 
in Version 4 (see RFC 4122). There is no need for monitoring the uniqueness,
 the chance of generating two UUIDs having the same value is about 6 x 10^-11.

The optional attribute "parent-uuid" is used whenever a task is changed. It is
an pointer to the original task-uuid. This is usefull to generate a version tree
in a later progress.

The task itself must have an attribute “lang” which specifies the natural language
used. The description, title etc should be written in this language. The content
of the “lang” attribute must comply with the IETF BCP 47, RFC 4647 and
ISO 639-1:2002 standards.

### The description part

```xml
    <xs:element name="description" type="xs:string" />
```

An instance of this element contains the task description as text. A
subset of HTML is allowed (see Appendix A).

### The proglang part

```xml
    <xs:element name="proglang">
      <xs:complexType>
         <xs:simpleContent>
             <xs:extension base="xs:string">
                 <xs:attribute name="version" type="xs:string" use="required"/>
             </xs:extension>
         </xs:simpleContent>
      </xs:complexType>
    </xs:element>
```

An instance of this element contains the programming/modelling/query
language to which this task applies. A valid list of values is specified
in Appendix B. The “version” attribute specifies which version of the
language was used in the creation of the task. (The task is guaranteed
to work with that version – any other requirements about version
compatibility must be checked externally.)  The “version” must be
entered as a “point” separated list of up to four unsigned integers.

### The submission-restrictions part

```xml
<xs:complexType name="submission-restrictions">
	<xs:choice>
		<xs:element name="archive-restrictions" type="tns:archiverestr-type"/>
		<xs:element name="files-restrictions" type="tns:file-restr-type"/>
		<xs:element name="regexp-restrictions" type="tns:file-regexp-restr-type"/>
	</xs:choice>
</xs:complexType>
```

An instance of this element can specify restrictions for the upload of
(solution) files - by default there are no restrictions. There is a choose
between three possible restrictions types. 
- [archive-restriction](#archive-restriction) 
- [file-restriction](#file-restriction) 
- [regexp-restriction](#regexp-restriction)

All restriction types have two optional attributes

-  “max-size” specifies the maximum size of a file in bytes which should be
   accepted. Systems which have a stronger limit of the file size should print a
   warning to the importing user. If this attribute is missing, a system default
   value will be used.
-  "mimetype-regexp" specifies the mimetype by regular expression of files the
   system should accept (specified regexp language in [regexp-language-restriction]
     (#regexp-language-specification))

```xml
<xs:attributeGroup name="maxsize-attr">
	<xs:attribute name="max-size" type="xs:positiveInteger" use="optional"/>
</xs:attributeGroup>
<xs:attributeGroup name="mimetype-attr">
	<xs:attribute name="mime-type-regexp" type="xs:string" use="optional"/>
</xs:attributeGroup>
```
#### Archive restriction

```xml
<xs:complexType name="archiverestr-type">
	<xs:choice>
		<xs:element name="unpack-files-from-archive-regexp" type="xs:string" maxOccurs="1"/>
		<xs:element name="file-restrictions">
			<xs:complexType>
				<xs:choice minOccurs="0" maxOccurs="unbounded">
					<xs:element name="required">
						<xs:complexType>
							<xs:attributeGroup ref="tns:mimetype-attr"/>
							<xs:attribute name="path" use="required"/>
						</xs:complexType>
					</xs:element>
					<xs:element name="optional">
						<xs:complexType>
							<xs:attributeGroup ref="tns:mimetype-attr"/>
							<xs:attribute name="path" use="required"/>
						</xs:complexType>
					</xs:element>
				</xs:choice>
			</xs:complexType>
		</xs:element>
	</xs:choice>
	<xs:attributeGroup ref="tns:maxsize-attr"/>
	<xs:attributeGroup ref="tns:mimetype-attr"/>
	<xs:attributeGroup ref="tns:archive-attr"/>
</xs:complexType>
...
<xs:attributeGroup name="archive-attr">
	<xs:attribute name="unpack-files-from-archive" type="xs:boolean" use="optional" default="false"/>
	<xs:attribute name="allowed-archive-filename" type="xs:string" use="optional"/>
</xs:attributeGroup>
```

 - "unpack-files-from-archive" specifies if uploaded archives (zip/jar) should
   be unpacked automatically. If it is set to *false,* no extraction takes place
   and the archive is used as it is.
 - the filename of the uploaded archive must match "allowed-archive-filename"
 
There is a choice for handling the file restrictions.

1. by regexp: The archive may contain many files. The regular expression specifies, which files will be extracted from the archive

```xml
   <tns:submission-restrictions>
        <tns:archive-restrictions max-size="[size in bytes]" mime-type-regexp="[mimetype regexp]" unpack-files-from-archive="[true/false]">
              <tns:unpack-files-from-archive-regexp>regular expression</tns:unpack-files-from-archive-regexp>
        </tns:archive-restrictions>
    </tns:submission-restrictions>
``` 

 -   “unpack-files-from-archive-regexp” holds a regular expression that
     controls which files are automatically extracted. Only matching files (the
     whole path of the zip-items matches with “/” as path separator) are
     extracted from the archive (specified regexp language in [regexp-language-restriction]
     (#regexp-language-specification)).
 
2. by filenames: The archive must or may contain files as specified by the following file restrictions

```xml
   <tns:submission-restrictions>
        <tns:archive-restrictions max-size="[size in bytes]" mime-type-regexp="[mimetype regexp]" unpack-files-from-archive="[true/false]">
            <tns:file-restrictions>
                <tns:required path="[full path to file]" mime-type-regexp="[mimetype regexp]" />
                <tns:optional path="[full path to file]" mime-type-regexp="[mimetype regexp]"/>
            </tns:file-restrictions>
        </tns:archive-restrictions>
    </tns:submission-restrictions>
```

- "required" the archive must contain a file with specified "path" (rooted at the archive root) and optional "mime-type-regexp". Otherwise the submission should be rejected.
- "optional" the archive may contain a file with specified attributes

#### File restriction

```xml
<xs:complexType name="file-restr-type">
	<xs:all>
		<xs:element name="required" minOccurs="0" maxOccurs="1">
			<xs:complexType>
				<xs:attributeGroup ref="tns:mimetype-attr"/>
				<xs:attributeGroup ref="tns:maxsize-attr"/>
				<xs:attribute name="filename" use="required"/>
			</xs:complexType>
		</xs:element>
		<xs:element name="optional" minOccurs="0" maxOccurs="1">
			<xs:complexType>
				<xs:attributeGroup ref="tns:mimetype-attr"/>
				<xs:attributeGroup ref="tns:maxsize-attr"/>
				<xs:attribute name="filename" use="required"/>
			</xs:complexType>
		</xs:element>
	</xs:all>
</xs:complexType>
```
A submission must or may consist of files as specified by the file restrictions. A submission should be rejected, if it does not match the restrictions.

```xml
   <tns:submission-restrictions>
            <tns:file-restrictions>
                <tns:required path="[full path to file]" mime-type-regexp="[mimetype regexp]" max-size="[size in bytes]"/>
                <tns:optional path="[full path to file]" mime-type-regexp="[mimetype regexp]" max-size="[size in bytes]"/>
            </tns:file-restrictions>
    </tns:submission-restrictions>
```

- "required" the submission must have a file with specified "path" (rooted at the archive root). Otherwise the submission should be rejected.
- "optional" the submission may have a file with specified attributes.

#### Regexp restriction
```xml
<xs:complexType name="file-regexp-restr-type">
	<xs:simpleContent>
		<xs:extension base="xs:string">
			<xs:attributeGroup ref="tns:maxsize-attr"/>
			<xs:attributeGroup ref="tns:mimetype-attr"/>
		</xs:extension>
	</xs:simpleContent>
</xs:complexType>
```

A submission must consist of one or several files, where all file names must adhere to the regular expression.

```xml
   <tns:submission-restrictions>
            <tns:regexp-restriction mime-type-regexp="[mimetype regexp]" max-size="[size in bytes]">regular expression</tns:regexp-restriction>
    </tns:submission-restrictions>
```

- "regexp-restriction" holds a regular expression of the filenames (only the filename, without path) which the system should accept. Regular expressions can contain less than/greater than signs. CDATA is allowed.

#### Regexp language specification

TODO!!!

### The files part

```xml
    <xs:element name="files">
        <xs:complexType>
            <xs:sequence minOccurs="0" maxOccurs="unbounded">
                <xs:element ref="tns:file"/>
            </xs:sequence>
        </xs:complexType>
        <xs:key name="fileid">
             <xs:selector xpath="tns:file"/>
             <xs:field xpath="@id"/>
        </xs:key>
    </xs:element>
```

The files element contains 0 or more file elements. A file element is
used to attach files to a task. Files can be external or embedded into
the XML file.

### The file element

```xml
    <xs:element name="file">
      <xs:complexType>
         <xs:simpleContent>
             <xs:extension base="xs:string">
                 <xs:attribute name="id" type="xs:string" use="required"/>
                 <xs:attribute name="filename" type="xs:string" use="optional"/>
                 <xs:attribute name="comment" type="xs:string" use="optional"/>
                 <xs:attribute name="class" use="required">
                     <xs:simpleType>
                         <xs:restriction base="xs:string">
                             <xs:enumeration value="template"/>
                             <xs:enumeration value="library"/>
                             <xs:enumeration value="inputdata"/>
                             <xs:enumeration value="instruction"/>
                             <xs:enumeration value="internal-library"/>
                             <xs:enumeration value="internal"/>
                         </xs:restriction>
                     </xs:simpleType>
                 </xs:attribute>
                 <xs:attribute name="type" default="embedded">
                    <xs:simpleType>
                         <xs:restriction base="xs:string">
                              <xs:enumeration value="file"/>
                              <xs:enumeration value="embedded"/>
                         </xs:restriction>
                    </xs:simpleType>
                 </xs:attribute>
             </xs:extension>
         </xs:simpleContent>
      </xs:complexType>
    </xs:element>
```

The file element includes or links a single file to a task. Each
instance/file must have a (task) unique string in its “id” attribute (in
order to reference this file within this task) and has to be classified
using the “class” attribute with one of the following values:

-   “template”: The file is a template for students to be used as a
    starting point for their solution.
-   “library”: The file is a library to be used by students (and might
    also be required for tests).
-   “inputdata”: The file contains data which the algorithm of the
    students should work with.
-   “instruction”: The file contains further instructions for handling
    the task, e.g. an UML activity diagram.
-   “internal-library”: This file is not visible for students and holds
    libraries which are required for processing the tests within the system.
-   “internal”: This file is not visible for students and holds files
    which are required for processing the task/tests within the system.


The information about how a file is used in a test is supplied by the
test-configuration which uses a file (via its ID). Further details can
be provided in the optional “comment” attribute. The file itself can be
embedded into the XML (recommended for shorter plain text files) or can
be included in the ZIP-archive (recommended for binary files). If a file
is embedded, the “type” attribute must be set to “embedded” and the text
content of the element is the file content. The “filename“ attribute can
be set to define a filename (e.g., for Java classes where the class
name must be equal to the filename, it can also include a relative
path). This attribute defines the name the file will have when it is
executed. If the name is unimportant, a default name can be used. If a
file is not embedded, the “type” attribute must be set to “file” and the
text content of the element contain the filename within the task ZIP
archive (which can be different from the filename attribute).



### The external-resources part

```xml
    <xs:element name="external-resources">
      <xs:complexType>
           <xs:sequence minOccurs="0" maxOccurs="unbounded">
                <xs:element ref="tns:external-resource"/>
           </xs:sequence>
      </xs:complexType>
      <xs:key name="external-resourceid">
           <xs:selector xpath="tns:external-resource"/>
           <xs:field xpath="@id"/>
      </xs:key>
    </xs:element>
```

The external-resources element contains 0 or more external-resource elements. An external-resource element is
used to refer to a resource that is neither embedded nor directly attached to the task.

### The external-resource element

```xml
    <xs:element name="external-resource">
      <xs:complexType>
           <xs:sequence>
               <xs:element ref="tns:description" minOccurs="0"/>
               <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded" processContents="lax"/>
           </xs:sequence>
           <xs:attribute name="id" type="xs:string" use="required"/>
           <xs:attribute name="reference" type="xs:string" use="optional"/>
      </xs:complexType>
    </xs:element>
```

Normally task files should be self-contained, but in rare cases the use of external resources is unavoidable for fulfilling or grading the task.  The external-resource element basically contains a reference to that kind of resources. In its simplest form, the resource is identified by an identifier contained in the “reference” attribute. More complicated references can be specified in child elements of any namespace.

The idea behind external resources is that sometimes huge files needed by a grader cannot be bundled reasonably with the task itself. A different use case is that the task needs to reference the latest version (in contrast to a specific version) of a file, which changes frequently and which is hosted in a central repository like maven central repository. A third use case is a task, for which the grader needs to access a web service while grading. In all three use cases, attaching or embedding the file with the task is disadvantageous or even (for the web service) impossible. In these or in similar cases, the task.xml may reference the external resource by a unique name or any other identifier. Examples are the name or URL of a widely known database dump (e. g. ftp://ftp.fu-berlin.de/pub/misc/movies/database/) or the name and version number of a library (e. g. urn:mvn:groupId=org.mockito:artifactId=mockito-core:packaging=jar:version=1.9.5) or a web service URL. The semantic and the format of references is not defined by this exchange format. It could be a URL, URN or any other identifier. The semantic largely depends on how the grader interprets the identifier of the resource and where the referenced data can be cached. Also the exchange format does not define the distribution mechanism of resources (e. g. push from LMS to grader, active pull by the grader, etc.). Taken to the extreme, an external-resource element may mean, that the administrator has to install some software, service, or data in a grader specific location, before the task can be used to grade submissions. The identifier therefore does not need to be in a machine readable format. Further details about the semantic of the external-resource can be provided in the optional “description” element. The description may provide instructions in natural language how to install the required resource prior to grading so that the grader can interpret and resolve the reference attribute successfully when grading a submission.

Identifying external resources can get quite complicated. Example: a SQL grader is able to work with either an Oracle or a MySQL database server. When working with Oracle the task must provide version A of a database dump, otherwise it must provide version B. Additionally there might be a range or a list of version numbers, that will fulfil the needs of the task. If the grader has one of these versions cached already, there is no need to install a different version. In order to cover the requirements of any grader, a task may specify detailed grader specific dependency and versioning information in child elements from any namespace.

Each external resource element can be identified by its mandatory “id” attribute and is referenced by the test-configuration (see below in the test section).

A task that references at least one external resource is not "self-contained" anymore. The author of the task should take care that the referenced resources are publicly available. Otherwise the task won't be reusable in other than the author's application context.


### The model-solutions part

```xml
    <xs:element name="model-solutions">
        <xs:complexType>
            <xs:sequence maxOccurs="unbounded">
                <xs:element ref="tns:model-solution"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
```

The model solutions element is used to provide one or more solutions of
the task. For each model-solution a new model-solution element is added.

### The model-solution element

```xml
    <xs:element name="model-solution">
        <xs:complexType>
             <xs:sequence>
                <xs:element ref="tns:filerefs"/>
             </xs:sequence>
             <xs:attribute name="id" type="xs:string" use="required"/>
             <xs:attribute name="comment" type="xs:string" use="optional"/>
        </xs:complexType>
        <xs:unique name="model-solutionid">
             <xs:selector xpath="tns:model-solution"/>
             <xs:field xpath="@id"/>
        </xs:unique>
    </xs:element>
```

The model-solution element links one single model-solution to a task. Each
instance/solution must have a (task) unique string in its “id” attribute.
The model-solution must refer to one or more files using the filerefs/fileref tag.
The optional attribute “comment” can be used for additional
information, for example if more than one model solution is provided it
can be explained why there are several solutions.

### The tests part

The tests element is used to provide automatic checks and tests for the
task. More specific information about the test XML is provided in the
[second section](#test-section) of this paper.

### The grading-hints element

```xml
    <xs:element name="grading-hints">
        <xs:complexType>
         <xs:sequence>
             <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
         </xs:sequence>
        </xs:complexType>
    </xs:element>
```

An instance of this element holds information on how the creator of the
task intended the grading process. This element contains plain text or
arbitrary elements in different namespaces. This field is mainly
intended to support an exchange of grading ideas and also to allow tasks
to be exported and imported again from one system to another.

### The meta-data element

```xml
    <xs:element name="meta-data">
        <xs:complexType>
         <xs:sequence>
             <xs:element ref="tns:title"/>
             <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
         </xs:sequence>
        </xs:complexType>
    </xs:element>
```

The meta-data element holds a namespace for the meta-data of each
system. Because meta-data are already standardized in other systems, it
was decided not to formalize them as part of this specification. Only
meta-data relevant for the whole task should be entered here. Meta-data
that is specific to an individual test should be entered in the
test-meta-data element.

## Test section

### XML Specification

The general structure of the test description is given as follows:

```xml
    <tns:tests>
        <tns:test id="unique ID" validity="">
            <tns:title></tns:title>
            <tns:test-type></tns:test-type>
            <tns:test-configuration>
                <tns:filerefs>
                    <tns:fileref></tns:fileref>
                </tns:filerefs>
                <tns:externalresourcerefs>
                    <tns:externalresourceref></tns:externalresourceref>
                </tns:externalresourcerefs>
                <tns:test-meta-data />
            </tns:test-configuration>
        </tns:test>
    </tns:tests>
```

The corresponding XML schema for the test XML structure is:

```xml
    <xs:element name="tests">
        <xs:complexType>
            <xs:sequence minOccurs="0" maxOccurs="unbounded">
                <xs:element ref="tns:test"/>
            </xs:sequence>
        </xs:complexType>
        <xs:unique name="testids">
             <xs:selector xpath="tns:test"/>
             <xs:field xpath="@id"/>
        </xs:unique>
    </xs:element>
```

### The test element

```xml
    <xs:element name="test">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="tns:title"/>
                <xs:element ref="tns:test-type"/>
                <xs:element ref="tns:test-configuration"/>
            </xs:sequence>
            <xs:attribute name="validity" use="optional" default="1.00">
                <xs:simpleType>
                    <xs:restriction base="xs:decimal">
                        <xs:totalDigits value="3"/>
                        <xs:fractionDigits value="2"/>
                        <xs:minInclusive value="0"/>
                        <xs:maxInclusive value="1.00"/>
                    </xs:restriction>
                </xs:simpleType>
            </xs:attribute>
            <xs:attribute name="id" use="required" type="xs:string"/>
        </xs:complexType>
    </xs:element>
```

The test element has a required attribute “id” and an optional attribute
“validity”. The optional attribute “validity” is used by some systems
(such as Vips) for tests which only partially verify the solution code.

### The title element

```xml
    <xs:element name="title" type="xs:string"/>
```

The title element is used to provide a short and clear name for the test
that can be displayed to students as part of their results. It should be
noted that the title does not have a language attribute because it is
assumed that the title is written in the same natural language as
specified for the task itself.

### The test-type element

```xml
    <xs:element name="test-type" type="xs:string"/>
```

Examples of values are: java-syntax, unittest. A list of allowed
entries is specified in Appendix C.

### The test-configuration part

```xml
    <xs:element name="test-configuration">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="tns:filerefs" minOccurs="0"/>
                <xs:element ref="tns:externalresourcerefs" minOccurs="0"/>
                <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
                <xs:element ref="tns:test-meta-data" minOccurs="0"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
```
	
The test-configuration contains all parameters which are needed for
configuring this specific test. The test-configuration should either
contain a files part or a code element which contains the actual code of
the test. It has sub-elements which are required, however, each test can
also have elements of its own namespace for test-type specific
configuration options.

### The filerefs part

```xml
    <xs:element name="filerefs">
        <xs:complexType>
            <xs:sequence maxOccurs="unbounded">
                <xs:element ref="tns:fileref"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
```

Several filerefs can be specified via fileref elements.

### The fileref element

```xml
    <xs:element name="fileref">
        <xs:complexType>
            <xs:attribute name="refid" type="xs:string" use="required"/>
        </xs:complexType>
    </xs:element>
```

The fileref element links a single file to a test based on the ID of the
file which has to be defined in task/files. The ID has to be entered as
the refid attribute.

### The externalresourcerefs part

```xml
    <xs:element name="externalresourcerefs">
        <xs:complexType>
            <xs:sequence minOccurs="0" maxOccurs="unbounded">
                <xs:element ref="tns:externalresourceref"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
```

Several externalresourcerefs can be specified via externalresourceref elements.

### The externalresourceref element

```xml
    <xs:element name="externalresourceref">
        <xs:complexType>
            <xs:attribute name="refid" type="xs:string" use="required"/>
        </xs:complexType>
    </xs:element>
```

The externalresourceref element links a single external-resource to a test based on the ID of the
external-resource which has to be defined in task/external-resources. The ID has to be entered as
the refid attribute.

### The test-meta-data element

```xml
    <xs:element name="test-meta-data">
        <xs:complexType>
         <xs:sequence>
             <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
         </xs:sequence>
        </xs:complexType>
    </xs:element>
```

The test-meta-data element holds a namespace for test-specific meta-data
of each system. This is particularly useful for attributes that are
required for ex- and import in one system but which are not relevant for
other systems.

## Appendix A: Subset of HTML

<\!\-\- … \-\-\>, a, b, blockquote, br, p, sup, sub, center, div, dl, dd,
dt, em, font, h1, h2, h3, h4, h5, h6, hr, img, li, ol, strong, pre,
span, table, tbody, td, tr, th, tt and ul.

However, for div, b, br, dd, dt, dl, blockquote, p, sup, sub, h1, ...h6,
li, ol, hr, strong, pre, span, table, tbody, td, tr, th, tt and ul there
are no attributes allowed in order to avoid problems with different
layouts.

## Appendix B: List of programming languages

-   java
-   SQL
-   prolog

## Appendix C: List of test types

-   java-compilation
-   java-checkstyle
-   java-code-coverage-emma
-   java-findbugs
-   java-pmd
-   unittest (urn:proforma:tests:unittest:v1)
-   dejagnu
-   anonymity (heuristics for checking that students have not included
    their names in the code)

