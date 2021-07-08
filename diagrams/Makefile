PUML_DIR = $(PWD)/puml
PNG_DIR = $(PWD)/png
SVG_DIR = $(PWD)/svg

# The resolution for the PNG files
PNG_DPI = 150

PUML_FILES = $(shell find $(PUML_DIR) -type f -name '*.puml')
PNG_FILES := $(patsubst $(PUML_DIR)/%.puml, $(PNG_DIR)/%.png, $(PUML_FILES))
SVG_FILES := $(patsubst $(PUML_DIR)/%.puml, $(SVG_DIR)/%.svg, $(PUML_FILES))

$(SVG_DIR)/%.svg: $(PUML_DIR)/%.puml
	plantuml -tsvg -o $(dir $@) $^

svg: $(SVG_FILES)

$(PNG_DIR)/%.png: $(PUML_DIR)/%.puml
	plantuml -SDpi=$(PNG_DPI) -o $(dir $@) $^

png: $(PNG_FILES)

.PHONY : clean
clean :
	-rm -rf $(PNG_DIR)
	-rm -rf $(SVG_DIR)
