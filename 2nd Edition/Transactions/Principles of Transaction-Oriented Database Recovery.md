# Principles of Transaction-Oriented Database Recovery


Three sections. 
Section 1: A short description of what recovery is expected to accomplish and which notion of consistency we assume.

Section 2: Provides an implementational model for database systems (a mapping hierarchy of data types)

section 3: Key concepts of the framework, describing the db states after a crash, the type of log information required, and additional measures for facilitating recovery.

Points out that the contents of section 1 rely on the description of failure types and the concept of a transaction given by Gray et al (1981)

Points out that isolation as a concept was considered from the origins of this concept, as far back as the Bjork and Davies 'spheres of control' paper in 1973. Then, this idea was restricted to use in database systems by Eswaran et al (1976) and named as a "Transaction".
