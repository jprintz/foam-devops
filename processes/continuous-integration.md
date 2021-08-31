# Continuous Integration

Continuous Integration mainly consists of the following steps in the DevOps process loop:

* Plan
* Create
* Verify

In my mind there are two part of the CI process that are quite distinct. The first one is the continuous integration of code into your main repository and test so that no new additions during development creates issues.
The second one is the continuous integration of the information received from the monitoring of your deployed application into what direction you want focus your development. Maybe everything works perfectly, then you have room for more features. Maybe your need to work on performance?

On a more practical level CI means that you want to test each individual branch of code **before** merging to a main branch that then gets tested further. In order for this to work the developers working on each branch need to write [[unit-tests]] in order to validate their code. If no unit tests exist or if they aren't passed, notify the developer(s) and **don't** merge into the main branch.
