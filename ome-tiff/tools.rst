Extracting, processing and validating OME-XML
=============================================
 
Extracting the OME-XML from an OME-TIFF file
--------------------------------------------

If you install the :bf_doc:`Bio-Formats command line
tools <users/comlinetools/>`, you can
produce a nicely formatted OME-XML string from an OME-TIFF file with:

::

    $ tiffcomment file.ome.tif | xmlindent

Alternatively, if you have ImageMagick installed, one easy way to extract
the OME-XML embedded in the TIFF headers is to use it from the command
line:

::

    $ identify -verbose

If you are working in C/C++, we recommend the open source
`LibTIFF <http://www.libtiff.org/>`_ library or the
:cpp_downloads:`OME Files C++ implementation <>`.

If you are looking for a solution in Java, there are several options.
Bio-Formats can read OME-TIFF files, as well as convert from many
third-party formats into OME-TIFF format—see the :doc:`example source code
page <code>` for specific examples. Alternatively, the open
source `ImageJ <https://imagej.net/ij/>`_ application reads
multi-page TIFF files, storing the TIFF comment into the associated
FileInfo object's "description" field.

Processing an OME-XML block
---------------------------

If the XML was stored without line breaks, it can still be difficult to
read after being extracted. There are several solutions to this problem,
such as using an XML viewer or editor (web browsers work well), or
processing the XML with a SAX or DOM library.

On most Linux distributions, you can install the libxml package and use
the ``xmllint`` program:

::

    $ xmllint --format file.xml

Here is a Perl script that uses
`XML::LibXML <https://metacpan.org/pod/XML::LibXML>`_ to
"pretty print" an XML document with appropriate whitespace:

::

    formatxml.pl

    use XML::LibXML;
    $file = $ARGV[0];
    $parser = XML::LibXML->new(); die "Cannot create XML parser" unless defined $parser;
    $parser->validation(0);
    if (defined $file) { $doc = $parser->parse_file($file); }
    else { $doc = $parser->parse_fh(STDIN); } print $doc->toString(1);

Unfortunately, both ``xmllint`` and the above Perl script can be somewhat
fragile; if there are any errors or abnormalities in the XML, they
generally fail to produce any indentation. Thus, we have also written
some Java code to do the same thing; just download the 
:bf_doc:`Bio-Formats command line tools <users/comlinetools/>` and run:

::

    $ xmlindent file.xml

Another option is to feed the XML into our 
:doc:`OME-XML Java library </ome-xml/java-library>`, which provides methods 
for querying and manipulating the OME-XML (using DOM and SAX). This library is 
what Bio-Formats uses to work with OME-XML.

Validating OME-XML
------------------

We have created a command line tool in Java for validating OME-XML, and
included it as part of the Bio-Formats :bf_downloads:`bftools.zip download
<>`. Please refer to the :bf_doc:`Bio-Formats command line tools
<users/comlinetools/>` documentation
for more details but, in brief, you download and unzip the tools to produce a
collection of command line scripts for Unix/Mac and batch files for Windows.
The two commands we will use are:

.. glossary::

    xmlvalid 
        A command line XML validation tool

    tiffcomment
        Extracts the OME-XML block in an OME-TIFF file from the
        comment in the TIFF's first IFD entry.

All scripts require :file:`bioformats_package.jar` to be downloaded into the
same directory as the command line tools. Then to validate an OME-XML file
:file:`sample.ome` use:

::

    $ xmlvalid sample.ome

This validates the XML directly.

Then to validate an OME-TIFF file :file:`sample.ome.tif` use:

::

    $ tiffcomment sample.ome.tif | xmlvalid

This extracts the OME-XML from the TIFF then passes it to the validator. 
Typical successful output is:

::

    $ ./xmlvalid sample.ome
    Parsing schema path
    http://www.openmicroscopy.org/Schemas/OME/2010-06/ome.xsd
    Validating sample.ome
    No validation errors found.
    $

If any errors are found they are reported. When correcting errors, it is 
usually best to work from the top of the file as errors higher up can cause 
extra errors further down. In this example the output shows 3 errors but there 
are only 2 mistakes in the file.

::

    $ ./xmlvalid broken.ome
    Parsing schema path
    http://www.openmicroscopy.org/Schemas/OME/2010-06/ome.xsd
    Validating broken.ome
    cvc-complex-type.4: Attribute 'SizeY' must appear on element 'Pixels'.
    cvc-enumeration-valid: Value 'Non Zero' is not facet-valid with respect 
       to enumeration '[EvenOdd, NonZero]'. It must be a value from the enumeration.
    cvc-attribute.3: The value 'Non Zero' of attribute 'FillRule' on element 
       'ROI:Shape' is not valid with respect to its type, 'null'.
    Error validating document: 3 errors found
    $


.. Also available is a web-based `OME-XML
   validator <http://validator.openmicroscopy.org.uk/>`_ for checking files
   in OME-XML or OME-TIFF formats, including:
   
   -  conformation to the OME-XML schema
   -  listing missing internal references
   -  listing external references
   -  correct TiffData block usage
   -  frame counts in TIFF files

Alternatively you can chose a freely available online validator 
to validate your OME-XML blocks e.g. one from `www.utilities-online.info <http://www.utilities-online.info/xsdvalidation>`_ .

Another option is to use a commercial XML application such as Turbo XML
to work with and validate your OME-XML documents.

