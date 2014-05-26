.. link: 
.. description: 
.. tags: 
.. date: 2014/05/11 17:45:31
.. title: ¿Se puede testear un print?
.. slug: se-puede-testear-un-print  

Mientras sprinteabamos `pbt <http://www.github.com/pebete/pbt>`_, en la ultima pycamp se me planteo una duda existencial: ¿Se puede testear un print?. La respuesta me llego de la mano de nessita(La guru del testing de la comunidad python-argentina) y me trajo muchas gratas sorpresas.

Parches para todos
------------------

Supongamos que queremos testear una funcion que hace algo tan simple como esto:

.. code-block:: 
    
    def printpow(i):
        print "my value is %d" i**2


