# Copyright 2014 The Cockroach Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied. See the License for the specific language governing
# permissions and limitations under the License. See the AUTHORS file
# for names of contributors.
#
# Author: Spencer Kimball (spencer.kimball@gmail.com)
# Author: Andrew Bonventre (andybons@gmail.com)

# Ensure we only have one entry in GOPATH
GOPATH := $(shell pwd)/../../../../..

PROTOC      := $(GOPATH)/bin/protoc
PROTOS      := $(sort $(wildcard *.proto))
PROTO_GO    := $(PROTOS:.proto=.pb.go)
GOGO_PROTOS := $(GOPATH)/src/github.com/gogo/protobuf/gogoproto/gogo.proto
SOURCES     := $(PROTOS:.proto=.pb.cc) github.com/gogo/protobuf/gogoproto/gogo.pb.cc
HEADERS     := $(PROTOS:.proto=.pb.h) github.com/gogo/protobuf/gogoproto/gogo.pb.h

PROTO_PATH := ../..:$(GOPATH)/src/github.com/gogo/protobuf:$(GOPATH)/src/github.com/gogo/protobuf/protobuf
PROTOC_GEN_GOGO := $(GOPATH)/bin/protoc-gen-gogo

# Protoc is very picky about paths. It doesn't like '..' in any of the input filenames,
# and generally expects paths and package names to match (this is especially important
# when compiling the proto files to python). We must work from a few levels higher in
# the tree so we can use the package name "cockroach.proto". (go is fine with a package
# name like "proto" but that won't work for other languages where package names come
# from a global namespace).
#
# For c++, it generates a directory structure mirroring the package structure
# (and these directories must be in the include path), but cgo can only compile
# a single directory so we symlink the generated pb.cc files into the storage/engine
# directory.
#
# The gogoprotobuf customtype and unmarshaller extensions do not play nicely with integer
# types. Manually wack the generate code to make it compile.
all:
	(cd ../.. && $(PROTOC) --proto_path=$(PROTO_PATH)   --cpp_out=cockroach/storage/engine       $(GOGO_PROTOS))
	(cd ../.. && $(PROTOC) --proto_path=.:$(PROTO_PATH) --cpp_out=cockroach/storage/engine       $(filter %api.proto %data.proto %internal.proto %config.proto %errors.proto, $(PROTOS:%=cockroach/proto/%)))
	(cd ../.. && $(PROTOC) --proto_path=.:$(PROTO_PATH) --plugin=$(PROTOC_GEN_GOGO) --gogo_out=. $(PROTOS:%=cockroach/proto/%))
	(cd ../storage/engine && ln -sf gogoproto/gogo.pb.cc .)
	(cd ../storage/engine && ln -sf cockroach/proto/*.pb.cc .)
	sed -E 's/(Node|Store)ID \|= \(int32/\1ID |= (\1ID/g' < config.pb.go > config.pb.go.new && \
	  mv config.pb.go.new config.pb.go
	sed -E 's/(Node|Store)ID \|= \(int32/\1ID |= (\1ID/g' < data.pb.go > data.pb.go.new && \
	  mv data.pb.go.new data.pb.go
	sed -E 's/(Node|Store)ID \|= \(int32/\1ID |= (\1ID/g' < log.pb.go > log.pb.go.new && \
	  mv log.pb.go.new log.pb.go
	sed -E 's/(Node|Store)ID \|= \(int32/\1ID |= (\1ID/g' < status.pb.go > status.pb.go.new && \
	  mv status.pb.go.new status.pb.go
