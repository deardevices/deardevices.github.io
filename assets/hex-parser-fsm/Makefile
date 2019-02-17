.phony: all
all: fsm.dot.png

%.dot.png : %.dot
	dot -Tpng $< > $@

clean:
	rm -f *.dot.png
