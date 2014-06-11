# Author:  Bruce Allen <bdallen@nps.edu>
# Created: 2/25/2013
#
# The software provided here is released by the Naval Postgraduate
# School, an agency of the U.S. Department of Navy.  The software
# bears no warranty, either expressed or implied. NPS does not assume
# legal liability nor responsibility for a User's use of the software
# or the results of such use.
#
# Please note that within the United States, copyright protection,
# under Section 105 of the United States Code, Title 17, is not
# available for any work of the United States Government and/or for
# any works created by United States Government employees. User
# acknowledges that this software contains work which was created by
# NPS government employees and is therefore in the public domain and
# not subject to copyright.
#
# Released into the public domain on February 25, 2013 by Bruce Allen.

# This file assists in building test programs.

lib_LTLIBRARIES = scan_hashid.la

#libscan_hashid_la_LDFLAGS = -static -avoid-version
scan_hashid_la_LDFLAGS = -export-dynamic -module -avoid-version

# for access to be13_api and dfxml
#AM_CPPFLAGS = -isystem${top_srcdir}/src/be13_api -isystem${top_srcdir}/src
AM_CPPFLAGS = -I${top_srcdir}/src/be13_api -I${top_srcdir}/src

scan_hashid_la_LADD = libhashdb.la

AM_LDFLAGS = -static

# actually, all of be13_api and some of dfxml are required
BE13_API_INCS = \
	bulk_extractor_i.h

scan_hashid_la_SOURCES = \
	$(BE13_API_INCS) \
	scan_hashid_plugin.cpp


.PHONY: 

