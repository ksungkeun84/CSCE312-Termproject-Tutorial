Stage4: Memory Stage
=====================

.. raw:: html

   <iframe width="700" height="400" src="https://www.youtube.com/embed/TJB2ORD2aNc?list=PLAN5AcM4p7jcTwCe-q-A6ziFdvkrXmnGe" title="8 mem" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

.. figure:: ./images/fig4.30.png

   Figure 4.30 Memory stage


.. list-table:: Control logic for Execution stage.

  * - instruction
    - Mem. read (1: read)
    - Mem. write (1: write)
    - Mem. addr (valE/valA)
    - Mem. Data (valA/valP)

  * - halt
    -
    -
    -
    -

  * - nop
    -
    -
    -
    -
  
  * - rrmovq rA, rB
    -
    -
    -
    -

  * - irmovq V, rB
    -
    -
    -
    -

  * - rmmovq rA, D(rB)
    -
    -
    -
    -

  * - mrmovq D(rB), rA
    -
    -
    -
    -

  * - OPq rA, rB
    -
    -
    -
    -

  * - jXX Dest
    -
    -
    -
    -

  * - cmovXX rA, rB
    -
    -
    -
    -

  * - call Dest
    -
    -
    -
    -

  * - ret
    -
    -
    -
    -

  * - pushq rA
    -
    -
    -
    -

  * - popq rA
    -
    -
    -
    -
