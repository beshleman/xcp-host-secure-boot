# Xen Host Secure Boot

This repository holds the present state of work towards enabling secure boot
XCP-ng.

## Overview

Supporting UEFI Secure Boot on XCP-ng today requires various changes to the
system.  First, patches supporting the unified image must be backported to the
XCP-ng Xen patch queue.  Due to shortcomings in the binutils version in the
XCP-ng build environment, a newer binutils build must be built and packaged.  A
second xen RPM called xen-unified, which has access to both the Xen binary
*and* the kernel (two seperate RPMs), is built and packaged and contains the
unified image.  Finally, a tool that supports the signing of RPMs in a way that
plays nicely with the Koji build system is underdevelop to sign the unified
image present in xen-unified package (and other packages if/when necessary).

### Dependencies

To see the RPM sources and tools source code, please refer to `.gitmodules`.

The XCP-ng build environment is based on the docker container tag
`centos:7.5.1804`.  Much of the tooling is old and needs updating to build Xen
as a unified image and to be signed for secure boot.

These dependencies, per RPM, include:

* xen-unified
  * binutils >= 2.36
* xen (unified binary patches)
  * binutils >= 2.36
* signing-service
  * sbsigntools
* sbsigntools
  * binutils >= 2.36
* binutils >= 2.36

### Progress

## xen-unified

The RPM to build the unified image is done.  This RPM combines the config, the
hypervisor, and the kernel into a single image.

## xen

All of the patches necessary to support the unified image have been backported
to Xen.  This gives Xen the ability to introspect and find the sections of its
image that contain the necessary boot modules.

## sbsigntools

This was a simple packaging task.  Package sbsigntools for the xcp-ng-env build
environment docker image.

## signing-service

A signing service was created that opens up an RPM, signs the relevant binaries
and repackages it into a replica of the original RPM (the same method used by
OpenSUSE, a repackaging perl script borrowed from OpenSUSE too).  But it is
unclear how to plug it into Koji reasonably (it turns out, the mechanism for
push post-build signed RPMs to the RPM repo is *only* for RPM signing, not
pushing RPMs generally.  If no way is found, then instead there may be a need
to build a service to tunnel a pesign socket to the signing machine so that the
signing may be done inside the RPM %build step.  TBD.
