Logisim Memory File generation using yo file
===============================================

This section describes how to create logisim memory file out of .yo file.
Let's suppose we have Y86-64 SEQ processor implemented with the logisim tool. Then, how can you run test programs?
In real computer system, we usually write a program with high level programming language such as C/C++ and compile it to make an executable file.
Then, operating system loads the executable to the main memory and the processor starts program execution.

Since we focus on the processor design so we simplify the above processes.

.. raw:: html

   <iframe width="700" height="400" src="https://www.youtube.com/embed/GjJf90EJU2k?list=PLAN5AcM4p7jcTwCe-q-A6ziFdvkrXmnGe" title="3 yo2mem" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

.. raw:: html

   <iframe width="700" height="400" src="https://www.youtube.com/embed/9L_6m7HtN1k?list=PLAN5AcM4p7jcTwCe-q-A6ziFdvkrXmnGe" title="3 yo2mem demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

1. Overall Steps
******************

::

    .ys --yas--> .yo --yo2mem--> .mem

Specifically, we rely on ``yas`` which is Y86-64 assembler. ``yas`` takes assembly code (.ys) as an input and outputs a memory file ``.yo`` that contains both machine code and data.
However, Y86-64 SEQ implemented with logisim tool cannot directly understand ``yo`` file.
Therefore, we are supposed to make a convertor from .yo to something that logisim can understand.

2. .yo file format
**********************
.. code-block:: bash
  :caption: asum.yo

                                | # Execution begins at address 0
    0x000:                      | 	.pos 0
    0x000: 30f40002000000000000 | 	irmovq stack, %rsp  	# Set up stack pointer
    0x00a: 803800000000000000   | 	call main		# Execute main program
    0x013: 00                   | 	halt			# Terminate program
                                |
                                | # Array of 4 elements
    0x018:                      | 	.align 8
    0x018: 0d000d000d000000     | array:	.quad 0x000d000d000d
    0x020: c000c000c0000000     | 	.quad 0x00c000c000c0
    0x028: 000b000b000b0000     | 	.quad 0x0b000b000b00
    0x030: 00a000a000a00000     | 	.quad 0xa000a000a000
                                |
    0x038: 30f71800000000000000 | main:	irmovq array,%rdi
    0x042: 30f60400000000000000 | 	irmovq $4,%rsi
    0x04c: 805600000000000000   | 	call sum		# sum(array, 4)
    0x055: 90                   | 	ret
                                |
                                | # long sum(long *start, long count)
                                | # start in %rdi, count in %rsi
    0x056: 30f80800000000000000 | sum:	irmovq $8,%r8        # Constant 8
    0x060: 30f90100000000000000 | 	irmovq $1,%r9	     # Constant 1
    0x06a: 6300                 | 	xorq %rax,%rax	     # sum = 0
    0x06c: 6266                 | 	andq %rsi,%rsi	     # Set CC
    0x06e: 708700000000000000   | 	jmp     test         # Goto test
    0x077: 50a70000000000000000 | loop:	mrmovq (%rdi),%r10   # Get *start
    0x081: 60a0                 | 	addq %r10,%rax       # Add to sum
    0x083: 6087                 | 	addq %r8,%rdi        # start++
    0x085: 6196                 | 	subq %r9,%rsi        # count--.  Set CC
    0x087: 747700000000000000   | test:	jne    loop          # Stop when 0
    0x090: 90                   | 	ret                  # Return
                                |
                                | # Stack starts here and grows to lower addresses
    0x200:                      | 	.pos 0x200
    0x200:                      | stack:

```.yo`` file includes memmory contents followed by its corresponding assembly code in the input ``.ys`` file.
In memory contents, it starts with start memory address followed by machine code or data. We just need memory contents for the logisim memory file.

3. Logisim Memory File
************************
Logisim has its own file format that represents the content of memory. There are some versions of the memory file.
If you are interested in logisim's memory file format in detail, please refer to the logisim reference.
In our term project, we are stick to use ``v3.0 hex words addressed``.


4. YO2MEM pytyon script
************************

.. code-block:: python
  :linenos:

    import sys
    import argparse

    def parse(line):
        data = line.split('|')[0].strip()
        if ':' not in data:
            return None, None

        addr = data.split(':')[0].strip()[2:]
        data = data.split(':')[1].strip()

        if len(data) == 0:
            return None, None

        assert(len(data) % 2 == 0)
        num_bytes = int(len(data) / 2)

        data_split = ''
        for offset in range(num_bytes):
            #print('offset: %d, idx: %d' % (offset, offset*2))
            data_split += data[offset*2:offset*2+2] + ' '
        return addr, data_split[:-1]

    def add(memory, addr, data):
        #assert(len(data) % 2 == 0)
        addr = int(addr, 16)
        memory[addr] = data

    def translate(yo, mem):
        #print("yo: %s, mem: %s" % (yo, mem))
        memory = []
        with open(yo, 'r') as f:
            for line in f:
                addr, data = parse(line)
                if addr is not None:
                    #print('addr: %s data: %s(%d)' % (addr, data, len(data)))
                    memory.append((addr, data))

        f = open(mem, 'w')
        f.write('v3.0 hex words addressed\n')
        for addr, data in memory:
            f.write('%s: %s\n' % (addr, data))
        f.close()
        print("Translated %s file to memory file logisim-evolution. Find %s." % (yo, mem))

    def main():
        parser = argparse.ArgumentParser(
                prog="yo2mem",
                description="y86-64 object file to logisim-evolution memory file translator",
                formatter_class=argparse.ArgumentDefaultsHelpFormatter
        )
        parser.set_defaults(func=lambda x: parser.print_help())
        parser.add_argument('yo', action='store', type=str, help="input yo file")
        parser.add_argument('mem', action='store', type=str, help="output memory file")

        args = parser.parse_args(sys.argv[1:])
        translate(args.yo, args.mem)

    if __name__ == "__main__":
        sys.exit(main())



This pythone script take two command line arguments as follows:

.. code-block:: bash

  ❯ python yo2mem -h
  usage: yo2mem [-h] yo mem

  y86-64 object file to logisim-evolution memory file translator

  positional arguments:
    yo          input yo file
    mem         output memory file

  optional arguments:
    -h, --help  show this help message and exit

The generated memory file will be used for both instruction and data memory.

.. code-block:: bash

  v3.0 hex words addressed
  000: 30 f4 00 02 00 00 00 00 00 00
  00a: 80 38 00 00 00 00 00 00 00
  013: 00
  018: 0d 00 0d 00 0d 00 00 00
  020: c0 00 c0 00 c0 00 00 00
  028: 00 0b 00 0b 00 0b 00 00
  030: 00 a0 00 a0 00 a0 00 00
  038: 30 f7 18 00 00 00 00 00 00 00
  042: 30 f6 04 00 00 00 00 00 00 00
  04c: 80 56 00 00 00 00 00 00 00
  055: 90
  056: 30 f8 08 00 00 00 00 00 00 00
  060: 30 f9 01 00 00 00 00 00 00 00
  06a: 63 00
  06c: 62 66
  06e: 70 87 00 00 00 00 00 00 00
  077: 50 a7 00 00 00 00 00 00 00 00
  081: 60 a0
  083: 60 87
  085: 61 96
  087: 74 77 00 00 00 00 00 00 00
  090: 90

