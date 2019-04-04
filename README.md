# csx730-more-paging Another Paging Activity

Most modern systems support a large virtual address space (e.g., `1ul << 64`).
In such an environment, the page table itself becomes excessively large.
One way to deal with this scenario is to use a multi-level paging algorithm,
sometimes referred to as **hierarchical paging**.

### Getting Started

1. Form into **small groups of two or three** people. These instructions assume that at least one group
   member is logged into the Nike.

1. Use Git to clone the repository for this exercise onto Nike into a subdirectory called
   `csx730-morepaging`:

   ```
   $ git clone https://github.com/cs1730/csx730-more-paging.git
   ```

1. Change into the `csx730-more-paging` directory that was just created and look around.
   There should be a couple different files already present, including a magic `Makefile`
   that will compile, assemble, and link stand-alone C files individually.

### Activity Questions

This activity is open book, open notes, and asking your instructor questions is allowed.
You may find Ch. 9 of _Operating System Concepts_ by Silberschatz, Gagne, and Galvin
useful as a reference.

1. Modify `SUBMISSION.md` to include the name, UGA ID number, course number (4730 or 6730)
   for each group member. Then, **sign the piece of paper that your instructor has at the front
   of the room.**

1. Under "Questions" in `SUBMISSION.md`, provide answers to the following
   questions, assuming you are on a 64 bit system with a 4096 byte page size:

   1. What is the size of the set of all virtual addresses?
      Show your work. You may express the solution in pseudo-C code.

   1. Assuming a single page table, how many pages are theoretically possible?
      Show your work. You may express the solution in pseudo-C code.

   1. Assuming a single page table, how big would that table need to be,
      in the following units: B and GB? Show your work. Please provide the exact
      integer values.

   1. Assume that each virtual address is broken up in to three parts:

      ```
      |-------------------------------------------------------|
      | table0 page number | table1 page number | page offset |
      |-------------------------------------------------------|
      ```

      If the number of bits reserved for `table0` and `table1` are
      equal, then how big would `table0` need to be, in the
      following units: B and KB? Show your work. Please provide the exact
      integer values.

   1. Assume that each virtual address is broken up in to five parts:

      ```
      |-------------------------------------------------------|
      | table0 PN | table1 PN | table2 PN | table3 PN | PO    |
      |-------------------------------------------------------|
      ```

      If the number of bits reserved for each table is equal,
      then how big would `table0` need to be, in the following
      units: B and KB? Show your work. Please provide the exact
      integer values.

**CHECKPOINT**

1. In `paging.c`, create a `struct` for an `addr_type` that uses
   [bit fields](https://en.cppreference.com/w/c/language/bit_field)
   to define the various parts of four-table, hierarchical virtual
   memory address:

   * Each field should be of type `unsigned long`.
     This is allowed by GCC even in strictly conforming code.

   * The variable for each page number should be `pn0`, `pn1`, etc.,
     and the page offset variable should be called `off`.

   * In this scenario, GCC will pack the members of the `struct`
     in order from the lowest order bits to the highest order bits
     (i.e., from right to left).

1. For convenience, also create a `typedef` for an `addr_t` type based
   on `struct addr_type`.

1. Under "Struct Size" in `SUBMISSION.md`, please indicate what
   the size of the `struct` is, in bytes.

1. In `main`, initialize an `addr_t` with some page number values
   and an offset value. For example, you might assign each part
   a value that is one less than a power of two so that they're
   easier to spot:

   ```c
   addr_t virt = { 1, 3, 7, 15, 31 };
   ```

   Then, `make` the executable and use GDB to
   confirm that the binary representation of the `addr_t` contains
   the values in their appropiate locations within the type. You
   may findf `x/gt` useful for examining the address in GDB:

   ```
   (gdb) x/gt &virt
   0x7fffffffe158:  0000000111110000000001111000000000011100000000000110000000000001
   ```

1. Repeat the last step for a couple different virtual addresses
   to confirm that your `struct` is defined correctly.

**CHECKPOINT**

1. In `paging.c`, declare a [`union`](https://en.cppreference.com/w/c/language/union)
   called `entry_type` that has the following overlapping members:

   * `addr_t frame`
   * `union entry_type * next`

1. For convenience, also create a `typedef` for an `entry_t` type based
   on `union entry_type`.

1. In `paging.c`, declare a page table called `table0` with
   the maximum number of entries based on the bit-width of
   `pn0` in `addr_t`. Each element of the array should be of
   type `entry_t`. In `main`, initialize the entries in the table
   to `0`.

   **CHECK YOURSELF:** Is the size of the table what you thought it would be?

1. Next, implement the following function:

   * `addr_t virt_to_phys(addr_t virt);` -- returns the physical address for a
     given virtual address based on the page table.

     * To compute the address, you will need to follow the table to the
       next table, etc., until you get to the final table, which will give
       you the frame number. In an ideal scenario where all the tables
       already exists, you might do something similar to the following:

       ```c
       addr_t v = // some address
       addr_t f = table0[v.pn0].next[v.pn1].next[v.pn2].next[v.pn3].frame;
       addr_t p = f * 4096 + v.off;
       ```

       Here is an illustration of the general process:

       ```
             table0        table1        table2        table3
            |------|  /~~>|------|  /~~>|------|  /~~>|------|
       [pn0]|    ~~+~/    |      |  |   |      |  |   |      |
            |      | [pn1]|    ~~|~/    |      |  |   |      |
            |      |      |      | [pn2]|    ~~|~/    |      |
            |      |      |      |      |      | [pn3]|   20 |
            |------|      |------|      |------|      |------|
       ```

       In general, the first three tables provide pointers to the next table
       and the final table provides the actual frame number. Therefore,
       a value of `0` in the first three tables is considered a `NULL` pointer
       whereas a `0` in the final table would refer to the first frame in
       physical memory.

     * If the next table does not exist, then you should use `malloc(3)` to
       dynamically allocate the table and initialize its values. For this
       exercise, you may initialize the intermediate tables to and frame numbers
       such that no frames overlap between addresses. In a real operating system,
       `kmalloc` (i.e., the kernel's memory allocator) is usually used to create
       the subsequent tables. Here is an example:

       ```c
       if (table0[v.pn0].next == NULL) {
           table0[v.pn0].next = malloc(sizeof(entry_t) * count);
       } // if
       ```

1. In `main`, write code to test your implementation. Provide enough output to
   convince yourself and others that your implementation works. At a minumum,
   this should demonstrate the translation of virtual addresses that take
   similar and different paths through the tables.

**SUBMISSION**

1. **Before 11:55 PM on FRI**, double check that your group member names are listed
   in `SUBMISSION.md` as well as the piece of paper that your instructor has at the
   front of the room, then submit your activity attempt using the `submit` command.
   From the parent directory:

   ```
   $ submit csx730-more-paging csx730
   ```

1. Remember to share this directory with your group members!

<hr/>

[![License: CC BY-NC-ND 4.0](https://img.shields.io/badge/License-CC%20BY--NC--ND%204.0-lightgrey.svg)](http://creativecommons.org/licenses/by-nc-nd/4.0/)

<small>
Copyright &copy; Michael E. Cotterell and the University of Georgia.
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a> to students and the public.
The content and opinions expressed on this Web page do not necessarily reflect the views of nor are they endorsed by the University of Georgia or the University System of Georgia.
</small>
