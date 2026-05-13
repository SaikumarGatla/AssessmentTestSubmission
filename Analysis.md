# 1. What is the exact cause of ConcurrentModificationException in Java?
ConcurrentModificationException in Java is thrown when a fail-fast collection is
structurally modified while it is being iterated using an Iterator, enhanced for-loop,
or stream iteration, except through the Iterator's own remove() method.

Structural modifications include operations such as:
.add()
.remove()
resizing the collection

Internal Working of fail-fast Collections:
To identify the structural modification of Collection, there is a variable "modCount".
The value of modCount is updated each time a Collection is modified structurally.
If there is a difference in modCount and expectedModCount ConcurrentModificationException is thrown.

    //Sometime like:
    if(modCount != expectedModCount) {
            thrown new ConcurrentModificationException();
    }

In ArrayList, the iterator internally checks whether the collection's
modification count (modCount) changed during iteration.
If the collection is modified structurally during traversal,
the iterator detects the mismatch and throws ConcurrentModificationException.

----------------------------------------------------------------------------------------------------------------------------------------

# 2. What code pattern at line 142 most likely triggered this error?
The most likely code pattern is modifying the same ArrayList structurally while iterating it inside an enhanced for-loop.

    public void reproduceConcurrentModificationException(List<Transaction> transactions) {
    //        Code snippet 1:
        for (Transaction tx : transactions) {
            if (tx.isValid()) {
            transactions.remove(tx);
        }
    }

    //        OR:

    //        Code snippet 2:
    //        Iterator<Transaction> itr = transactions.iterator();
    //        while (itr.hasNext()) {
    //            Transaction tx = itr.next();
    //            transactions.remove(tx);
    //        }

    }

----------------------------------------------------------------------------------------------------------------------------------------

# 3. Provide the minimal code change (one or two lines) that resolves this safely
Preferred minimal fix:
Use Iterator.remove() instead of modifying the list directly during iteration.

Example:

        Iterator<Transaction> itr = transactions.iterator();
        while (itr.hasNext()) {
            Transaction tx = itr.next();

            if (tx.isInvalid()) {
                itr.remove(); // FIX: remove using iterator instead of list.remove()
            }
        }

Reason:
Iterator.remove() updates the iterator state safely without causing ConcurrentModificationException

Alternative approach:
A fail-safe collection like CopyOnWriteArrayList could avoid
ConcurrentModificationException, but that would be a broader design change
and not the minimal surgical fix expected for this task.
Especially because CopyOnWriteArrayList creates a new copy of the array on every write operation,
which is usually not ideal for transaction-processing systems with frequent modifications.


