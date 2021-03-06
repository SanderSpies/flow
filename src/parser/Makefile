# Copyright (c) 2013-present, Facebook, Inc.
# All rights reserved.

DIR:=$(dir $(abspath $(lastword $(MAKEFILE_LIST))))
TOP=$(DIR)../..

NATIVE_OBJECT_FILES=\
	hack/heap/hh_shared.o\
	hack/utils/files.o\
	hack/utils/nproc.o\
	hack/utils/realpath.o\
	hack/utils/sysinfo.o\
	hack/utils/priorities.o\
	hack/utils/win32_support.o

RUNNER_DEPS=\
	hack/heap\
	hack/utils\
	hack/utils/collections\
	hack/utils/hh_json\
	hack/third-party/core\
	src/common\
	src/parser

all: build-parser

clean:
	ocamlbuild -clean
	rm -f flow_parser.js tools/native_test_files

build-parser:
	ocamlbuild -no-links parser_flow.cmxa

js:
	ocamlbuild -use-ocamlfind -pkgs js_of_ocaml -build-dir _build flow_parser_js.byte
	js_of_ocaml --opt 3 -o flow_parser.js _build/flow_parser_js.byte
	rm _build/flow_parser_js.byte

test-js: build-parser js
	npm test

_build/src/parser/test/run_hardcoded_tests.native: build-parser
	cd $(TOP); \
	ocamlbuild -build-dir src/parser/_build \
		-ocamlc "ocamlopt -ccopt -DNO_LZ4" \
		$(NATIVE_OBJECT_FILES); \
	ocamlbuild \
		-build-dir src/parser/_build \
		$(foreach dir,$(RUNNER_DEPS),-I $(dir)) \
	 	-lib unix -lib str \
	 	-lflags "$(NATIVE_OBJECT_FILES)" \
		src/parser/test/run_hardcoded_tests.native

test-ocaml: _build/src/parser/test/run_hardcoded_tests.native
	$< test/hardcoded_tests.js

test: test-js test-ocaml

tools: native_test_files

native_test_files:
	ocamlbuild tools/native_test_files.native
	mv native_test_files.native tools/native_test_files

npm-publish:
	mv README.md README-backup.md
	mv README-npm.md README.md
	-npm publish
	mv README.md README-npm.md
	mv README-backup.md README.md
