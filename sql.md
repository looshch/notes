# Notes On SQL
* [Index](#index)

## Index
‘An index makes the query fast’
* an index is a distinct structure in the database that is built using the
  `create index` statement. It requires its own disk space and holds a copy of
  the indexed table data. Creating an index does not change the table data; it
  just creates a new data structure that refers to the table. The database
  combines two data structures: a doubly linked list and a search tree. The
  primary purpose of an index is to provide an ordered representation of the indexed
  data. It is, however, not possible to store the data sequentially because an
  insert statement would need to move the following entries to make room for the
  new one. Moving large amounts of data is very time- consuming so the insert
  statement would be very slow. The solution to the problem is to establish a
  logical order that is independent of physical order in memory. Databases use
  doubly linked lists to connect the so-called index leaf nodes. The index order
  is maintained on two different levels: the index entries within each leaf
  node, and the leaf nodes among each other using a doubly linked list
