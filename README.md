# BWAPP-build-image

This project builds the compute/nexus3_demo image and pushes it to the nexus3 registry.  It takes two parameters to allow you to build different versions:

owner_email:
This is used to label the owner of the image

nexus3_version:
The version of nexus3 defaults to 3.20.0, the other valid values include 3.10.0, and 3.0.0.