Download Link: https://assignmentchef.com/product/solved-csci243-homework6-load-module-analyzer
<br>
<h1></h1>

In our discussion of program translation, we discussed the typical contents found in an object or load module file. One of these is the <em>Executable and Linking Format</em> (ELF)<u><sup>1</sup></u>, which is used in many different operating systems (including Linux<u><sup>®</sup></u>). It is an extremely flexible format, and even allows the creation of object and load module files which contain executable code for multiple architectures.

For this assignment, you will work with a much simpler object module format. This will require that you read and process non-text, binary data, and that you understand how information can be “packed” into words so as to save space.

The object module format you will work with was designed for use with the assembler, linker, and simulator used in the CSCI-250 (Concepts of Computer Systems) course, which uses the MIPS R2000 processor as its example of CPU design and assembly language programming. It is significantly simpler than the ELF format, and was designed to be easy to work with while still providing enough information to allow linking of separately-translated source programs. Because that course uses the MIPS R2000 architecture, this suite of programs is called the R2K suite, and thus the object module format is the R2K module format.

The R2000 is a 32-bit CPU with a 4 byte addresses and instructions. You will find a description of the R2K object module format in a separate document: <a href="https://www.cs.rit.edu/~csci243/protected/homeworks/07/r2k.html">The R2K Ob</a><a href="https://www.cs.rit.edu/~csci243/protected/homeworks/07/r2k.html">j</a><a href="https://www.cs.rit.edu/~csci243/protected/homeworks/07/r2k.html">ect Module Format</a>.

<h1>Supplied Code</h1>

Create a subdirectory to contain your work for this assignment, and cd into that directory. Retrieve the materials for this homework using the command:

get  csci243  hw7

This will copy a number of files from the course account into your working directory, as follows:

exec.h      Declaration of relevant structures and some CPP macros to simplify some tasks header.mak For use with gmakemake

sample.asm A simple R2K assembly language program sample.obj The object file version of sample.asm sample.out The load module (i.e., linked executable) version of sample.asm stdout.1    Output from running the command “alm sample.out” stdout.2     Output from running the command “alm sample.obj”

You should not submit any of these files. A copy of exec.h will be provided by try when you submit your solution. The sample object and load module files are to be used when testing your program.

<h1>Activities</h1>

Based on the R2K format described in the document linked above, you are to write a program which analyzes R2K object modules and load modules. Your program will be invoked with one or more R2K modules on the command line:

alm  <em><u>file-1</u></em> [ <em><u>file-2</u></em> … ]

For each <em><u>file-i</u></em> on the command line, your program will produce a summary of the information found in that object or load module, as described below.

Unless otherwise indicated, all output from your program should be printed to stdout. The only output printed to stderr are error messages associated with opening or processing an R2K module.

<h2>Command Line Argument Verification</h2>

If your program is invoked with no command-line arguments, print this usage message to stderr and exit:

usage: alm file1 [ file2 … ]

<h2>File Processing</h2>

For each command-line argument, you must open the file for input, read its contents, and produce your report. If the open fails, use perror() to print an error message with the name of the file which could not be opened as the prefix, and move to the next command-line argument.

Assuming the file can be opened, you must first verify the magic number. If the magic number is not

0xface, print the message error: <em><u>file</u></em> is not an R2K object module (magic number 0x<em><u>XXXX</u></em>)

to the stderr, where <em><u>file</u></em> is the name of the file being checked, and <em><u>XXXX</u></em> is the magic number found in the file (printed as a four-digit hex value). After this, move to the next command-line argument.

Once you have verified that the file is an R2K module, produce your report. Print a separator line consisting of a twenty hyphen (‘-‘) characters, followed by one line that indicates whether this is an object module or a load module. For object modules, print:

——————–

File <em><u>name</u></em> is an R2K object module

where <em><u>name</u></em> is the name of the R2K module file. For load modules, print:

——————-File <em><u>name</u></em> is an R2K load module (entry point 0x<em><u>aaaaaaaa</u></em>)

where <em><u>aaaaaaaa</u></em> is the 32-bit entry point, printed as an eight-character hex value. After this, decode the version number and report it:

Module version:  2<em>y<u>yy</u></em>/<em><u>mm</u></em>/<em><u>dd</u></em>

where the values you print are the decoded fields from the version number field in the module header, with the month and day printed as two-digit values. (You will not be doing any processing of the flags field in the header.)

Next, you must print a summary of the sections which are present in the module and their sizes. If a section has a non-zero size, print the following line for it:

Section <em><u>name</u></em> is <em><u>n</u></em> <em><u>unit</u></em> long

where <em><u>name</u></em> is the section name (“text”, “rdata”, “sbss”, etc.), <em><u>n</u></em> is the number of things in it, and <em><u>unit</u></em> is either bytes or entries, depending on whether this section is just binary data (text and data sections, string table) or an array of elements (relocation, reference, and symbol tables).

You will not do any processing of text and data section contents in this program, although you will need to somehow skip over those sections in order to get to the following ones. See below for some suggestions on how to do this.

If the relocation table is not empty, print an introductory line followed by one line (indented by three spaces) for each entry in the table; e.g., for <em>N</em> entries:

Relocation information:

0x<em><u>aaaaaaa1</u></em> (<em><u>sect1</u></em>) type 0x<em><u>ttt1</u></em>

0x<em><u>aaaaaaa2</u></em> (<em><u>sect2</u></em>) type 0x<em><u>ttt2</u></em>

. . .

0x<em><u>aaaaaaaN</u></em> (<em><u>sectN</u></em>) type 0x<em><u>tttN</u></em>

where each <em><u>aaaaaaaa</u></em> is the 32-bit address field printed as an eight-digit hex value, each <em><u>sect</u></em> is the name of the section to which this entry belongs, and each <em><u>tttt</u></em> is the relocation type printed as a four-digit hex value. (Note that you are <em>not</em> interpreting the type codes, just printing them out.)

Similarly, if the reference table is not empty, print the reference table using the following format:

Reference information:

0x<em><u>aaaaaaa1</u></em> type 0x<em><u>ttt1</u></em> symbol <em><u>name1</u></em>

0x<em><u>aaaaaaa2</u></em> type 0x<em><u>ttt2</u></em> symbol <em><u>name2</u></em>

. . .

0x<em><u>aaaaaaaN</u></em> type 0x<em><u>tttN</u></em> symbol <em><u>nameN</u></em>

where each <em><u>aaaaaaaa</u></em> and <em><u>tttt</u></em> are as in the relocation output, and each <em><u>name</u></em> is the symbol name (printed from the string table). (Again, note that you are not interpreting the type codes, just printing them out.)

Next, do the same thing for the symbol table if it is not empty; print a symbol table report using the following format:

Symbol table:

value 0x<em><u>vvvvvvv1</u></em> flags 0x<em><u>fffffff1</u></em> symbol <em><u>name1 </u></em>   value 0x<em><u>vvvvvvv2</u></em> flags 0x<em><u>fffffff2</u></em> symbol <em><u>name2</u></em>

. . .

value 0x<em><u>vvvvvvvN</u></em> flags 0x<em><u>fffffffN</u></em> symbol <em><u>nameN </u></em>——————–

where each <em><u>vvvvvvvv</u></em> is the 32-bit value associated with the symbol (as an eight-digit hex value), each <em><u>ffffffff</u></em> is the 32-bit flag word for the symbol (also printed as an eight-digit hex value), and each <em><u>name</u></em> is the symbol name (printed from the string table). (Once more note that you are not interpreting the type codes or the flags, just printing them out.)

Finally, print a separator line and then move to the next command-line argument:

——————–

<h2>An Example</h2>

Consider the contents of the R2K load module supplied to you with the name sample.out, printed as a series of 32-bit hex values:

220fcefa 00000000 64004000 84000000 20000000 08000000 00000000 00000000

00000000 00000000 00000000 08000000 38000000 fcffbd23 0000bfaf 00000234

21408000 2148a000 01002a29 04004015 00008a8c 04008420 ffff2921 05001008

0000bf8f 0400bd23 0800e003 fcffbd23 0000bfaf 0010043c 00008434 0010013c 2000258c 0000100c 0000bf8f 0400bd23 0800e003 0dad0b00 0000a48f 0400a58f

0800a68f 0e00100c 00000000 20204000 11000234 0c000000 0a000000 14000000 0c000000 eb000000 4f2f0000 b6680100 efffffff 00000000 07000000 00000000 b1000000 2c004000 00000000 a3000000 20000010 05000000 b1400000 38004000 0b000000 a1000000 14004000 10000000 b1400000 00004000 15000000 67000000 11000000 1f000000 a2000000 00000010 19000000 b1400000 64004000 29000000

656e6f64 756f6300 6d00746e 006e6961 706f6f6c 6d757300 72726100 53007961

455f5359 32544958 725f5f00 5f5f6b32 72746e65 005f5f79

Note the byte ordering in the first 32-bit value, which contains both the magic number and version number fields. For comparison, here is the same module printed as individual bytes in hex:

fa ce 0f 22 00 00 00 00 00 40 00 64 00 00 00 84 00 00 00 20 00 00 00 08 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 08 00 00 00 38 23 bd ff fc af bf 00 00 34 02 00 00 00 80 40 21 00 a0 48 21 29 2a 00 01 15 40 00 04 8c 8a 00 00 20 84 00 04 21 29 ff ff 08 10 00 05 8f bf 00 00 23 bd 00 04 03 e0 00 08 23 bd ff fc af bf 00 00 3c 04 10 00 34 84 00 00 3c 01 10 00 8c 25 00 20 0c 10 00 00 8f bf 00 00 23 bd 00 04 03 e0 00 08 00 0b ad 0d 8f a4 00 00 8f a5 00 04 8f a6 00 08 0c 10 00 0e 00 00 00 00 00 40 20 20 34 02 00 11 00 00 00 0c 00 00 00 0a 00 00 00 14 00 00 00 0c 00 00 00 eb 00 00 2f 4f 00 01 68 b6 ff ff ff ef 00 00 00 00 00 00 00 07 00 00 00 00 00 00 00 b1 00 40 00 2c 00 00 00 00 00 00 00 a3 10 00 00 20 00 00 00 05 00 00 40 b1 00 40 00 38 00 00 00 0b 00 00 00 a1 00 40 00 14 00 00 00 10 00 00 40 b1 00 40 00 00 00 00 00 15 00 00 00 67 00 00 00 11 00 00 00 1f 00 00 00 a2 10 00 00 00 00 00 00 19 00 00 40 b1 00 40 00 64 00 00 00 29 64 6f 6e 65 00 63 6f 75 6e 74 00 6d 61 69 6e 00 6c 6f 6f 70 00 73 75 6d 00 61 72 72 61 79 00 53

59 53 5f 45 58 49 54 32 00 5f 5f 72 32 6b 5f 5f

65 6e 74 72 79 5f 5f 00

In this version, the individual bytes appear in the “correct” sequence. The first row of hex values are the first 16 bytes of the load module; here they are regrouped into sizes that match the first fields of the exec_t structure:

face 0f22 00000000 00400064 00000084

The first two bytes contain the 16-bit magic number, which is correct (0xface). The next two bytes contain the version number; the value is 0x0f22, which decodes to 2007/09/02. Following that are the flag word (0x00000000), the entry point (0x00400064 – because this is non-zero, this is a load module rather than an object module), and the first element of the data array (0x00000084, which is the size of the text segment, in bytes). Your program should produce this output to this point:

——————–

File sample.out is an R2K load module (entry point 0x00400064)

Module version:  2007/09/02

Collectively, here are the 13 32-bit words comprising the header block:

face0f22 00000000 00400064 00000084 00000020 00000008 00000000 00000000

00000000 00000000 00000000 00000008

00000038

From this, we see that sections 0 (text), 1 (rdata), 2 (data), 8 (symbol table), and 9 (string table) have nonzero sizes, and therefore only those sections will be reported as having contents:

Section text is 132 bytes long

Section rdata is 32 bytes long

Section data is 8 bytes long

Section symtab is 8 entries long

Section strings is 56 bytes long

The next 132 bytes comprise the text section:

23bdfffc afbf0000 34020000 00804021 00a04821 292a0001 15400004

8c8a0000 20840004 2129ffff 08100005 8fbf0000 23bd0004 03e00008 23bdfffc afbf0000 3c041000 34840000 3c011000 8c250020 0c100000 8fbf0000 23bd0004 03e00008 000bad0d 8fa40000 8fa50004 8fa60008 0c10000e 00000000 00402020

34020011 0000000c

Following this are the 32 bytes of the read-only data section:

0000000a 00000014 0000000c 000000eb 00002f4f 000168b6 ffffffef 00000000

Next are the 8 bytes of the data section:

00000007 00000000

Remember that your program is not looking at the contents of any text or data sections in the module file; however, you must move past them in order to get to the following sections. See below for some suggestions as to how to do this.

The relocation and reference sections are empty in this module, which means that immediately after the data section will be the symbol table. Each entry is 12 bytes long, and there are 8 entries:

000000b1 0040002c 00000000 000000a3 10000020 00000005 000040b1 00400038 0000000b 000000a1 00400014 00000010 000040b1 00400000 00000015 00000067 00000011 0000001f 000000a2 10000000 00000019 000040b1 00400064 00000029

Each group of three 32-bit words is a symbol table entry (shown here with the decimal equivalent of the string table index in parentheses):

<h3>                     Flags       Address          Index</h3>

000000b1 0040002c 00000000 (0)

000000a3 10000020 00000005 (5)

000040b1 00400038 0000000b (11)

000000a1 00400014 00000010 (16)

000040b1 00400000 00000015 (21)

00000067 00000011 0000001f (31)

000000a2 10000000 00000019 (25)

000040b1 00400064 00000029 (41)

The string table is the remaining 56 bytes of the module:

64 6f 6e 65 00 63 6f 75 6e 74 00 6d 61 69 6e 00 6c 6f 6f 70 00 73 75 6d 00 61 72 72 61 79 00 53

59 53 5f 45 58 49 54 32 00 5f 5f 72 32 6b 5f 5f

65 6e 74 72 79 5f 5f 00

Broken into NUL-terminated strings, here are the offsets into the table, the bytes of the string, and their ASCII equivalents:

From this, we will produce the symbol table report. Putting that together with the output produced so far, here is the complete output for this module:

——————–

File sample.out is an R2K load module (entry point 0x00400064)

Module version:  2007/09/02

Section text is 132 bytes long

Section rdata is 32 bytes long

Section data is 8 bytes long

Section symtab is 8 entries long

Section strings is 56 bytes long

Symbol table:    value 0x0040002c flags 0x000000b1 symbol done    value 0x10000020 flags 0x000000a3 symbol count    value 0x00400038 flags 0x000040b1 symbol main    value 0x00400014 flags 0x000000a1 symbol loop    value 0x00400000 flags 0x000040b1 symbol sum    value 0x00000011 flags 0x00000067 symbol SYS_EXIT2    value 0x10000000 flags 0x000000a2 symbol array    value 0x00400064 flags 0x000040b1 symbol __r2k__entry__ ——————–

<strong>Note above that there are 2 lines with a string of hyphens (‘-‘); one is at the start, and the other at the end. </strong>When the program processes multiple files on the command line, there will be 2 adjacent lines of hyphens between each file’s information.