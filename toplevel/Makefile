all: toplevel.js eval.js


HIGLO=$(shell ocamlfind query higlo.reason -format "-package %p -rectypes" 2> /dev/null)
DERIVING=$(shell ocamlfind query js_of_ocaml.deriving -format "%p" 2> /dev/null)

ifneq ($(HIGLO),)
CPPO_OPT+= -D higlo
endif

ifneq ($(DERIVING),)
DERIVING_SYNTAX = js_of_ocaml.deriving.syntax deriving.syntax.std
endif

ifneq ($(NO_SYNTAXES),1)
SYNTAXES= lwt.syntax js_of_ocaml.syntax $(DERIVING_SYNTAX)
endif

JSFILES= +weak.js +toplevel.js +dynlink.js +nat.js

PACKAGES= \
	lwt bigarray tyxml.functor \
	react reactiveData \
	js_of_ocaml $(DERIVING)\
	js_of_ocaml.tyxml \
	js_of_ocaml.toplevel \
	str \
	reason.lib \
	dynlink

ifeq ($(WITH_GRAPHICS),YES)
JSFILES += +graphics.js
PACKAGES += graphics js_of_ocaml.graphics
CPPO_OPT+= -D graphics
endif

#-jsopt "--pretty"
MKTOP=jsoo_mktop -verbose $(SAFESTRING) \
	${addprefix -top-syntax , ${SYNTAXES}} \
	-dont-export-unit gc \
	-jsopt "--disable shortvar" \
	${addprefix -export-package , ${PACKAGES}} \
	${addprefix -export-unit , html_types html_sigs xml_wrap } \

TOPLEVEL_NAME=toplevel
TOPLEVEL_OBJS=colorize.cmo indent.cmo toplevel.cmo
$(TOPLEVEL_NAME).js: $(TOPLEVEL_OBJS) examples.ml test_dynlink.cmo
	$(MKTOP) \
	$(OCPINDENT) $(HIGLO) \
	$(TOPLEVEL_OBJS) \
	${addprefix -jsopt , ${JSFILES}} \
	${addprefix -jsopt , -I ./ --file examples.ml} \
	${addprefix -jsopt , -I ./ --file test_dynlink.cmo} \
	-package base64 \
	-o $(TOPLEVEL_NAME).byte

EVAL_NAME=eval
EVAL_OBJS=eval.cmo
$(EVAL_NAME).js: $(EVAL_OBJS)
	$(MKTOP) $(EVAL_OBJS) \
	${addprefix -jsopt , ${JSFILES}} \
	-o $(EVAL_NAME).byte

%.cmis.js:
	jsoo_mkcmis $*

server.%: server.ml
	ocamlbuild -use-ocamlfind -package findlib,cohttp.lwt server.$*

indent.cmo: indent.cmi
colorize.cmo: colorize.cmi

%.ml: %.cppo.ml
	cppo ${CPPO_OPT} $< -o $@

%.cmi:%.mli
	ocamlfind ocamlc -c -package js_of_ocaml,js_of_ocaml.tyxml $<
%.cmo:%.ml
	ocamlfind ocamlc -c -syntax camlp4o $(SAFESTRING) \
		-package js_of_ocaml.syntax,lwt,str,reason.lib,js_of_ocaml.tyxml,js_of_ocaml.toplevel \
		-package base64 \
		-I camlp4 \
		$(HIGLO) \
		$< -c $@
clean::
	rm -f *.cm[io] *.byte *.native *.js
