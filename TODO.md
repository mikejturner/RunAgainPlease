* Improve message when rap finds unsubbed token. It could point or color the unsubbed token to make a distinction between others on the line that do get subbed.
* Make the rap_basename user-definable
* If the input(body) is empty then don't write it to disk.
* Add support for command-line options using `docopt`
* Add support for a progress bar using `progressbar2`
* Allow items in list to include `,`. Escaping?
* Allow user to reference external scripts
* Change the range style back to start:stop:step
* Allow running till a threshold met?
* Allow the user to choose the shell (but default to bash if none specified)

Tricky
------

* Allow calculations non-locally e.g. supercomputers, clusters
* Report failures of calculations (only get script return code at present)
