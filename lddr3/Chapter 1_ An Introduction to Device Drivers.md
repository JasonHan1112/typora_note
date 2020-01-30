# Chapter 1: An Introduction to Device Drivers
-  we like this word because it emphasizes that the role of a device driver is providing mechanism, not policy.
    - The distinction between mechanism and policy is one of the best ideas behind the Unix design. Most programming problems can indeed be split into two parts: “what capabilities are to be provided” (the mechanism) and “how those capabilities can be used” (the policy). If the two issues are addressed by different parts of the program, or even by different programs altogether, the software package is much easier to develop and to adapt to particular needs.
-  Drivers of this sort not only work better for their end users, but also turn out to be easier to write and maintain as well. Being policy-free is actually a common target for software designers.
-  split view of the kernel
![353ebb41.png](attachments\353ebb41.png)