OME-TIFF example source code for common operations
==================================================

This section discusses some common operations related to the OME-TIFF
format, and provides example source code in Java for performing them. We
strongly recommend you read and understand the :doc:`specification` 
before attempting to deploy any of the following code.

-  Extracting a TIFF comment – demonstrates how to extract the OME-XML
   comment from an OME-TIFF file.
-  Modifying a TIFF comment – demonstrates how to modify an OME-XML
   comment within an OME-TIFF file.
-  Converting other formats to OME-TIFF – demonstrates how to convert
   third-party formats to OME-TIFF.

The :bf_doc:`Bio-Formats library <about/>`
provides a lot of functionality related to OME-XML and OME-TIFF, to ease
the burden of file format handling and conversions. Rather than offer
source code that performs all of the above actions on its own, we
instead offer examples that utilize Bio-Formats, for more robust and
succinct operation.

Extracting a TIFF comment
-------------------------

To extract a comment from a TIFF file without the aid of a TIFF library,
the following steps are required:

#. Read in the 8-byte header.
#. Determine if the file is a valid TIFF, and if so, whether its byte
   order is big endian or little endian. Bytes 0–1 must equal "II"
   (0x4949, little endian, "Intel") or "MM" (0x4D4D, big endian,
   "Motorola").
#. Check TIFF version.  Bytes 2–3 must equal 42 (0x2A) or 43 (0x2B)
   with the proper endianness, which are the TIFF specification or the
   newer BigTIFF specification, respectively.
#. If the TIFF version is 0x2A, offsets are 4 bytes (32-bit).  If the
   TIFF version is 0x2B, bytes 4–5 are the size of offsets in bytes
   (should be 0x0008 for 8-byte 64-bit offsets, but could be
   different), and bytes 6-7 are padding (should be 0x0000).
#. Determine the byte offset into the file of the first IFD. For TIFF
   version 0x2A, this information is stored in bytes 4–7, with the
   proper endianness.  For TIFF version 0x2B, this information is
   stored starting at byte 8, sized according to the specified offset
   size and with the proper endianness.  This will be bytes 8–15 for
   8-byte offsets.
#. Skip to the first IFD, and read in the IFD's header.
#. Iterate through the directory entries looking for the
   ImageDescription (270) tag.
#. Jump to the offset given by the ImageDescription entry, and read the
   number of bytes indicated by the entry.
#. Convert the bytes to an ASCII string.

The :bf_doc:`Bio-Formats command line
tools <users/comlinetools/>` include a
program, tiffcomment, that performs these steps using the
getComment(String) method of 
:bf_source:`TiffParser <components/formats-bsd/src/loci/formats/tiff/TiffParser.java>`.
You can produce a nicely formatted OME-XML string from an OME-TIFF file
with:

::

    tiffcomment file.ome.tif | xmlindent

.. seealso::

    `BigTIFF file format specification <https://web.archive.org/web/20240706160214/https://www.awaresystems.be/imaging/tiff/bigtiff.html>`__

Modifying a TIFF comment
------------------------

Modifying a TIFF comment can be tricky because the length of the altered
OME-XML string is unlikely to be the same as before. As such, the IFD's
ImageDescription directory entry must be updated to reflect the new byte
count. In addition, if the string is longer than before, it will no
longer fit at its old offset, unless the comment was at the end of the
file, so the entry's offset might need to change as well.

The ``TiffSaver.overwriteIFDValue()`` method within Bio-Formats, efficiently 
alters a directory entry with a minimum of waste. The count field of the entry 
is intelligently updated to match the new length. If the new length is longer 
than the old length, it appends the new data to the end of the file and 
updates the offset field; if not, or if the old data is already at the end of
the file, it overwrites the old data in place.

The following program extracts comments from TIFF files, prompts the
user to alter the comments on the command line, and writes updated
comments back to the files. It requires the
:bf:`Bio-Formats library <>`.

:bf_source:`EditTiffComment.java <components/formats-gpl/utils/EditTiffComment.java>`

The comment string is acquired using ``new TiffParser(f).getComment()``, and
updated with 

::

    TiffSaver saver = new TiffSaver(f);
    RandomAccessInputStream in = new RandomAccessInputStream(f);
    saver.overwriteComment(in, xml);

To quickly edit an OME-TIFF files comment on the command line use
``tiffcomment -edit filename.ome.tiff`` from the 
:bf_doc:`Bio-Formats command line tools <users/comlinetools/>`.

Converting other formats to OME-TIFF
------------------------------------

One of the major goals of Bio-Formats is to standardize the metadata
from all supported third-party formats into OME-XML. Doing so makes
conversion to OME-TIFF very straightforward—just write the pixels to
TIFF however you want (e.g. with libtiff), and store the converted
OME-XML metadata into the TIFF comment. The complicated part is doing
the conversion from proprietary third-party metadata into OME-XML—a task
that Bio-Formats greatly simplifies.

The following program converts the files given on the command line into
OME-TIFF format. It requires the :bf:`Bio-Formats <>` and :doc:`OME-XML
Java </ome-xml/java-library>` libraries.

:bf_source:`ConvertToOmeTiff.java <components/formats-gpl/utils/ConvertToOmeTiff.java>`

The code functions by creating an ImageReader for reading the input
files' image planes sequentially, and an OMETiffWriter for writing the
planes to OME-TIFF files on disk. The OME-XML is generated by attaching
an OMEXMLMetadata object to the reader, such that when each file is
initialized, the object is automatically populated with the converted 
metadata. The OMEXMLMetadata object is then fed to the OMETiffWriter, which 
extracts the appropriate OME-XML string and embeds it into the OME-TIFF file 
properly.

While our ultimate goal is for the Bio-Formats metadata conversion
facility to be a reference implementation for conversion of third-party
formats into OME-XML and OME-TIFF, please be aware that the current code
is a work in progress. We would greatly value suggestions and assistance
regarding the OME-XML conversion relating to any specific format. If
there is any metadata missing or converted incorrectly, please let us
know.

.. seealso::
    
    :bf_doc:`Exporting raw pixel data to OME-TIFF files <developers/export2.html>` 
    and :bf_doc:`Converting files from FV1000 OIB/OIF to OME-TIFF <developers/conversion.html>`
