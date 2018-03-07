# UCBOOPExample

Disclaimer: This code is not optimized and has not undergone any significant refactoring. The author recognizes several places where more efficient data structures and algorithms can be used to make the workflow more concise and DRY.

This coding sample demonstrates the use of multiple Javascript objects, both created and derived from JSON HTTP Responses.

APIFunctions.js is a created class within a wider Node.js on Express application which automatically creates reports by cross referencing data between two systems.

APiFunctions.js is an object-based, compound class of constructor functions which create database objects. APIFunctions.js has five external libraries, all available on npm and two internal libraries; one of which is included in the repo. 'functionalAreaMap' is a Javascript object exported as a module for ease of maintenance as an example of explicit object creation and usage as a map. As an export, APIFunctions.js returns an object of functions and mongoose models for use in the main method routes.

Object manipulation is carried through the use of attribute additon in methods such as getProjectorData(url) [line 88] and getProjectorBurnReport(url) [line 239]. 

If you would like to see the main method, ejs classes, or anything else regarding this project -- please let me know.

