.phony: all
all: graph1.dot.png

%.dot.png : %.dot
	dot -Tpng $< > $@

clean:
	rm -f *.dot.png
