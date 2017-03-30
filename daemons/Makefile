#
#
# $Id: Makefile,v 1.6 2005/05/02 00:24:57 root Exp $
#
INCLUDE=-I/usr/src/linux/include -I../dev
CC=gcc
LD=gcc
CFLAGS=-D_REENTRANT -static $(INCLUDE)
LDFLAGS=pthread


daemons := pld cm11ad plusbd


all: $(daemons)

cm11ad: main.o cm11a_xcvr.o
	gcc -Wall -o $@ $+ -lpthread

pld: main.o pl_xcvr.o
	gcc -Wall -o $@ $+ -lpthread

plusbd: main.o plusb_xcvr.o
	gcc -Wall -o $@ $+ -lpthread

cm17ad: main.o cm17a_xcvr.o
	gcc -Wall -o $@ $+ -lpthread

clean:
	rm -f $(daemons)
	rm -f *.o
	rm -f .depend
	rm -f .depend.bak

