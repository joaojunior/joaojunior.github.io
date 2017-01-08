Callbacks in Cplex
==================

I will try to use `LazyConstraintCallback <https://www.ibm.com/support/knowledgecenter/SSSA5P_12.6.0/ilog.odms.cplex.help/refpythoncplex/html/cplex.callbacks.LazyConstraintCallback-class.html>`_ 
to implement a `Branch and Cut <https://en.wikipedia.org/wiki/Branch_and_cut>`_ algorithm with 
`CPLEX <https://www-01.ibm.com/software/commerce/optimization/cplex-optimizer/>`_, but when I implemented and run this 
algorithm, it is very slow if I compare with version that don’t use callbacks in Cplex. It is because 
ControlCallback(LazyConstraintCallback is a ControlCallback) switch Cplex to sequential mode by default and turn off dynamic search.
I write this post, because I don’t find this information in Cplex’s documentation, I only found this information in this `technical forum <https://www.ibm.com/developerworks/community/forums/html/topic?id=55bc851b-cba5-446f-a6b9-696ad8ab0481>`_.

You can also read this in `Medium. <https://medium.com/@joaojunior.ma/callbacks-in-cplex-7e459571b007#.as9fz74xv>`_

.. author:: default
.. categories:: none
.. tags:: CPLEX, Branch And Cut, Optimization, Operational Research
.. comments::

