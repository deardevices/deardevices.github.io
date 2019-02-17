.phony: all
all: graph1.dot.png graph2.dot.png graph3.dot.png graph4.dot.png

%.dot.png : %.dot
	dot -Tpng $< > $@

clean:
	rm -f *.dot.png
