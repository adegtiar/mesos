# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
# 
#     http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License

BOOST = third_party/boost-1.37.0
PROTOBUF = third_party/protobuf-2.3.0
GLOG = third_party/glog-0.3.1
GMOCK = third_party/gmock-1.5.0
ZOOKEEPER = third_party/zookeeper-3.3.1/src/c
LIBPROCESS = third_party/libprocess
LIBEV = $(LIBPROCESS)/third_party/libev-3.8

# Add our own includes.
AM_CPPFLAGS = -I$(top_srcdir)/include

# Add boost to includes.
AM_CPPFLAGS += -I$(top_srcdir)/$(BOOST)

# Add protobuf to includes.
AM_CPPFLAGS += -I$(top_srcdir)/$(PROTOBUF)/src

# Add glog to includes.
AM_CPPFLAGS += -I$(top_srcdir)/$(GLOG)/src -I$(top_builddir)/$(GLOG)/src

# Add libprocess to includes.
AM_CPPFLAGS += -I$(top_srcdir)/$(LIBPROCESS)/include

# Add included ZooKeeper to includes (if necessary).
if WITH_INCLUDED_ZOOKEEPER
  AM_CPPFLAGS += -I$(top_srcdir)/$(ZOOKEEPER)/include \
                 -I$(top_srcdir)/$(ZOOKEEPER)/generated
endif

lib_LTLIBRARIES = libmesos.la

libmesos_la_SOURCES = mesos.pb.cc messages/messages.pb.cc		\
                      messages/log.pb.cc sched/sched.cpp		\
                      local/local.cpp log/replica.cpp			\
                      log/coordinator.cpp master/master.cpp		\
                      master/slaves_manager.cpp				\
                      master/frameworks_manager.cpp			\
                      master/allocator_factory.cpp			\
                      master/simple_allocator.cpp slave/slave.cpp	\
                      slave/isolation_module.cpp			\
                      slave/process_based_isolation_module.cpp		\
                      slave/reaper.cpp launcher/launcher.cpp		\
                      exec/exec.cpp common/fatal.cpp common/lock.cpp	\
                      detector/detector.cpp detector/url_processor.cpp	\
                      configurator/configurator.cpp			\
                      common/string_utils.cpp common/logging.cpp	\
                      common/date_utils.cpp common/tokenize.cpp		\
                      common/resources.cpp

libmesos_la_LIBADD = libbuild.la			\
  $(top_builddir)/$(PROTOBUF)/src/.libs/libprotobuf.la	\
  $(top_builddir)/$(GLOG)/.libs/libglog.la		\
  $(top_builddir)/$(LIBPROCESS)/libprocess.a		\
  $(top_builddir)/$(LIBEV)/.libs/libev.la

if WITH_INCLUDED_ZOOKEEPER
  libmesos_la_LIBADD += $(top_builddir)/$(ZOOKEEPER)/.libs/libzookeeper_mt.la
endif

noinst_LTLIBRARIES = libbuild.la
libbuild_la_SOURCES = common/build.cpp
libbuild_la_CPPFLAGS = -DBUILD_DATE="\"$$(date '+%Y-%m-%d %H:%M:%S')\"" \
                       -DBUILD_USER="\"$$USER\""                        \
                       -DBUILD_FLAGS="\"$(AM_CFLAGS) $(AM_CXXFLAGS)\""  \
                       $(AM_CPPFLAGS)
libbuild_la_DEPENDENCIES = FORCE

FORCE:

noinst_LTLIBRARIES = libwebui.la
libwebui_la_SOURCES = webui/master/swig/master_wrap.cpp	\
                      webui/slave/swig/slave_wrap.cpp

webui/master/swig/master_wrap.cpp: $(srcdir)/webui/master/swig/master.i $(srcdir)/master/state.hpp $(srcdir)/master/webui.hpp config/config.hpp
if WITH_WEBUI
	mkdir -p webui/master/swig
	$(SWIG) -c++ -python -outdir webui/master/swig -I$(srcdir) -o $@ $<
	cp -r webui/master/swig/master.py $(BINDIR)/webui/master/swig
else
	echo "" > $@
endif

webui/slave/swig/slave_wrap.cpp: $(SRCDIR)/webui/slave/swig/slave.i $(SRCDIR)/slave/state.hpp $(SRCDIR)/slave/webui.hpp $(SRCDIR)/slave/slave.hpp config/config.hpp $(BINDIR)/webui/slave/swig
ifeq ($(WITH_WEBUI),1)
	mkdir -p webui/slave/swig
	$(SWIG) -c++ -python -outdir webui/slave/swig -I$(SRCDIR) -o $@ $<
	cp -r webui/slave/swig/slave.py $(BINDIR)/webui/slave/swig
else
	echo "" > $@
endif



PROTOC = $(top_builddir)/$(PROTOBUF)/src/protoc

mesos.pb.cc: $(top_srcdir)/include/mesos/mesos.proto
	mkdir -p java/generated
	$(PROTOC) -I$(top_srcdir)/include/mesos \
            --cpp_out=. \
            --java_out=java/generated \
            $(top_srcdir)/include/mesos/mesos.proto

messages/messages.pb.cc: $(srcdir)/messages/messages.proto
	mkdir -p messages
	$(PROTOC) -I$(srcdir)/messages -I$(top_srcdir)/include/mesos \
            --cpp_out=messages $(srcdir)/messages/messages.proto

messages/log.pb.cc: $(srcdir)/messages/log.proto
	mkdir -p messages
	$(PROTOC) -I$(srcdir)/messages -I$(top_srcdir)/include/mesos \
            --cpp_out=messages $(srcdir)/messages/log.proto

CLEANFILES = mesos.pb.h mesos.pb.cc messages/messages.pb.h	\
             messages/messages.pb.cc messages/log.pb.h		\
             messages/log.pb.cc

bin_PROGRAMS = mesos-local
mesos_local_SOURCES = local/main.cpp
mesos_local_LDADD = $(PTHREAD_LIBS) libmesos.la

check_PROGRAMS = mesos-tests
mesos_tests_SOURCES = tests/main.cpp tests/utils.cpp			\
	              tests/master_tests.cpp				\
	              tests/resource_offer_tests.cpp			\
	              tests/resource_offer_reply_tests.cpp		\
	              tests/fault_tolerant_tests.cpp			\
	              tests/log_tests.cpp tests/resources_tests.cpp	\
	              tests/uuid_tests.cpp tests/external_tests.cpp	\
	              tests/sample_frameworks_tests.cpp			\
	              tests/configurator_tests.cpp			\
	              tests/string_utils_tests.cpp			\
	              tests/multimap_tests.cpp				\
	              tests/protobuf_io_tests.cpp			\
	              tests/lxc_isolation_tests.cpp
mesos_tests_CPPFLAGS = -I$(top_srcdir)/$(GMOCK)/gtest/include		\
                       -I$(top_srcdir)/$(GMOCK)/include $(AM_CPPFLAGS)
mesos_tests_LDADD = $(PTHREAD_LIBS) libmesos.la				 \
                    $(top_builddir)/$(GMOCK)/lib/.libs/libgmock.la	 \
                    $(top_builddir)/$(GMOCK)/gtest/lib/.libs/libgtest.la

TESTS = mesos-tests