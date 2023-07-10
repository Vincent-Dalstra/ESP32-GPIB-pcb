SHELL = /bin/sh

prj := $(basename $(wildcard *.kicad_pro))
out := Fab-Out
gout := ${out}/gerber

board-files := ${gout}/${prj}-CuTop.gtl
board-files += ${gout}/${prj}-PasteTop.gtp
board-files += ${gout}/${prj}-SilkTop.gto
board-files += ${gout}/${prj}-MaskTop.gts
board-files += ${gout}/${prj}-CuBottom.gbl
board-files += ${gout}/${prj}-PasteBottom.gbo
board-files += ${gout}/${prj}-SilkBottom.gbo
board-files += ${gout}/${prj}-MaskBottom.gbs
board-files += ${gout}/${prj}-EdgeCuts.gm1
board-files += ${gout}/${prj}.drl
board-files += ${gout}/${prj}-drl_map.pdf



ass-opts := --missingError --assembly 
ass-opts += --field LCSC --ignore JLCPCB_IGNORE --corrections JLCPCB_CORRECTION


# All: creates everything needed for fabrication.
.PHONY: all 
all: ${prj}.zip ${out}/bom.csv ${out}/pos.csv

# clean: Deletes all generated files.
.PHONY: clean
clean:
	rm -r ${out}/*

# Trim: Removes extraneous files; useful when packaging into release directory.
.PHONY: trim
trim:
	rm -r ${gout}

${prj}.zip : ${board-files}
	zip ${out}/${prj}.zip ${out}/gerber/*


# Create the Gerbers; checking for DRC ERRORS (not warning though) before we make it
${board-files}&: ${prj}.kicad_pcb
	kikit export gerber ${prj}.kicad_pcb ${out}/gerber/

# The Bill-of-materials is based on every schematic combined. The Position file is based on
# the pcb. Both are generated in a single command; hence the combined-target recipe.
${out}/bom.csv ${out}/pos.csv&: *.kicad_sch subsheet/* ${prj}.kicad_pcb
	kikit fab jlcpcb ${ass-opts} --schematic ${prj}.kicad_sch ${prj}.kicad_pcb ${out}/



# ----


# pinion: Open the interactive boardview
.PHONY: view_plotted view
view_plotted view&: pinion/plotted/spec.json
	pinion serve -b --directory pinion/plotted/
	
.PHONY: view_rendered
view_rendered&: pinion/rendered/spec.json
	pinion serve -b --directory pinion/rendered/

# Create the interactive boardview, using 2D image library for components
pinion/plotted/spec.json: pinion/spec.yaml
	pinion generate plotted --board ${prj}.kicad_pcb --specification pinion/spec.yaml pinion/plotted --pack --libs ~/Documents/Electronics/Kicad-libraries/PcbDraw-Lib/KiCAD-base/

# Create the interactive boardview, rendering from 3D models. Takes time.
pinion/rendered/spec.json: pinion/spec.yaml
	pinion generate rendered --board ${prj}.kicad_pcb --specification pinion/spec.yaml pinion/rendered --pack

# Create the default file from the template file, performing some post-processing
pinion/default.yaml: pinion/template.yaml
	./pinion/post-processing.sh pinion/template.yaml > pinion/default.yaml

# Create the template file. Update when the PCB file changes
pinion/template.yaml: ${prj}.kicad_pcb
	pinion template --board ${prj}.kicad_pcb --output pinion/template.yaml

