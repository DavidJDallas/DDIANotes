# Trade Offs in Data Systems Architecture

Different first chapter structure. Whereas the 1st edition focused more generally on what it means for a distributed system to be 'good', this explores a few different hot topics and brief considerations around trade-offs, giving pros and cons for when to use which.

Distinguishes in first few pages between compute-intensive and data-intensive. I think that Oneiro currently maybe blurs the lines between these two in terms of where the issues are, especially regarding long-running transactions, which seem primarily to be compute-intensive problems. It seems to be a problem not around horizontal scaling (since we're not running a distributed transaction), but one of needing more CPU power/splitting the transaction more.

## OLTP vs analytics

A brief discussion of the key differences (OLTP is for 'normal' development, reading and writing to the db and everyday application development. Analytics is for data analytics, data visualisation, business intelligence, data science stuff, where you'd be using exclusively Read queries, making frequently big requests as opposed to many smaller requests)

Also a useful distinction between Systems of Record and Derive data systems. The former is your source of truth; the latter is derived. For the first, think primary tables on your db; for the latter, think caches, derived tables (Views, e.g.), indexes. They can be wiped anbd safely regenerated.

## Cloud vs Self-Hosting

To decide between the two, a common rule of thumb is that 'things that are a core competency or a competitive advantage of your organisation should be done in-house, whereas things that are non-core, routine, commonplace should be left to a vendor' (p12).

In a traditional systems archtecture, the same computer is responsible for both storage (disk) and computation (CPU and RAM). In cloud native systems, the 2 responsibilities ahve become disaggregated.

The shift to cloud native has changed what operations teams look like, but operations teams remain and remain important.

## Distributed vs single-node systems
