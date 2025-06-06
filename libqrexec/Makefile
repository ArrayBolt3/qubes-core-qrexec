CC ?=gcc
VCHAN_PKG = $(if $(BACKEND_VMM),vchan-$(BACKEND_VMM),vchan)
CFLAGS += -I. -I../libqrexec -g -O2 -Wall -Wextra -Werror \
   $(shell pkg-config --cflags $(VCHAN_PKG)) -fstack-protector \
   -fstack-protector-strong -fPIC -std=gnu11 -D_POSIX_C_SOURCE=200809L \
   -D_GNU_SOURCE \
   -Wstrict-prototypes -Wold-style-definition -Wmissing-declarations \
   -fno-delete-null-pointer-checks -fvisibility=hidden \
   -Wvla -Wformat=2

LDFLAGS += -pie -Wl,-z,relro,-z,now -shared

SO_VER=4
VCHANLIBS := $(shell pkg-config --libs vchan)
LIBDIR ?= /usr/lib
INCLUDEDIR ?= /usr/include

_XENSTORE_H=$(shell ls /usr/include/xenstore.h 2>/dev/null)
ifneq "$(_XENSTORE_H)" ""
QUBES_CFLAGS += -DUSE_XENSTORE_H
endif


all: libqrexec-utils.so
libqrexec-utils.so.$(SO_VER): unix-server.o ioall.o buffer.o exec.o txrx-vchan.o write-stdin.o replace.o remote.o process_io.o log.o toml.o open_logger.o vchan_timeout.o
	$(CC) $(LDFLAGS) -Wl,-soname,$@ -o $@ $^ $(VCHANLIBS)

libqrexec-utils.so: libqrexec-utils.so.$(SO_VER)
	ln -sf $@.$(SO_VER) $@

%.a:
	$(AR) rcs $@ $^
clean:
	rm -f -- *.o *~ *.a *.so *.so.* *.o.dep *.gcda *.gcno

install:
	install -d -m 0755 $(DESTDIR)$(LIBDIR)
	install libqrexec-utils.so.$(SO_VER) $(DESTDIR)$(LIBDIR)
	ln -sf libqrexec-utils.so.$(SO_VER) $(DESTDIR)$(LIBDIR)/libqrexec-utils.so
	install -d -m 0755 $(DESTDIR)$(INCLUDEDIR)
	install -m 0644 libqrexec-utils.h $(DESTDIR)$(INCLUDEDIR)
	install -m 0644 qrexec.h $(DESTDIR)$(INCLUDEDIR)

%.o: %.c
	$(CC) $< -c -o $@ $(CFLAGS) -MD -MP -MF $@.dep

-include *.o.dep
