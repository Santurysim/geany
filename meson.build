project('geany', ['c', 'cpp'], version: '1.38.0', default_options: ['c_std=c99', 'cpp_std=c++11'])

fs = import('FS')
pymod = import('python')

conf = configuration_data()

docdir = get_option('datadir') / 'doc' / '@0@-@1@'.format(meson.project_name(), meson.project_version())

summary({'Install Geany in': get_option('prefix'),
		'Building Geany on': build_machine.system(),
		'Building Geany for': host_machine.system()})



cc = meson.get_compiler('c')
foreach h: ['fcntl.h', 'glob.h', 'stdlib.h', 'sys/time.h', 'errno.h', 'limits.h']
  cc.check_header(h, required: true)
endforeach


foreach h: ['fnmatch.h', 'direct.h', 'io.h', 'sys/dir.h']
  if cc.check_header(h)
	conf.set('HAVE_' + h.underscorefy().to_upper(), 1)
  endif
endforeach

conf.set('USE_STDBOOL_H', 1)
conf.set('CTAGS_LIB', 1)

foreach func: ['fgetpos', 'fnmatch', 'mkstemp', 'strerror', 'strstr', 'realpath']
  if cc.has_function(func)
	conf.set('HAVE_' + func.to_upper())
  endif
endforeach

foreach func: ['strcasecmp', 'stricmp']
  if cc.has_function(func):
	conf.set('HAVE_' + func.to_upper())
	break
  endif
endforeach

foreach func: ['strncasecmp', 'strnicmp']
  if cc.has_function(func):
	conf.set('HAVE_' + func.to_upper())
	break
  endif
endforeach

foreach func: ['truncate', 'ftruncate', 'chsize']
  if cc.has_function(func):
	conf.set('HAVE_' + func.to_upper())
	break
  endif
endforeach

conf.set10('HAVE_DECL___ENVIRON', cc.has_header_symbol('unistd.h', '__environ'))
conf.set10('HAVE_DECL_NSGETENVIRON', cc.has_header_symbol('crt_externs.h', '_NSGetEnviron'))

git = find_program('git', required: false)
if git.found()
 result = run_program(git, 'rev-parse', '--short', '--revs-only', 'HEAD') 
 if result.returncode() != 1
   revision = result.stdout().strip()
   add_project_arguments('-g', '-DGEANY_DEBUG', language: ['c', 'c++'])
 else
   revision = '-1'
 endif
else
  revision = '-1'
endif

summary({'Compiling Git revision': revision})

gtk_flags = []

if get_option('enable-gtk2')
  gtk = dependency('gtk+-2.0', version: '>=2.0')
  use_gkt2 = true
else
  gtk = dependency('gtk+-3.0', version: '>=3.0')
  gtk_flags += '-DGDK_DISABLE_DEPRECATION_WARNINGS'
  use_gtk2 = false
endif

glib = dependency('glib-2.0', version: '>=2.32')
gio = dependency('gio-2.0', version: '>=2.32')
gmodule_no_export = dependency('gmodule-no-export-2.0')

gtk_flags += '-DGLIB_VERSION_MIN_REQUIRED=GLIB_VERSION_2_32'

summary({'Using GTK version': gtk.version()})

gthread = dependency('gthread-2.0')

if get_option('disable-deprecated')
  gtk_flags += ['-DG_DISABLE_DEPRECATED', '-DGTK_DISABLE_DEPRECATED']
endif

# Check for binary relocation support

binreloc = get_option('enable-binreloc')

if binreloc.enabled()
  if fs.exists('/proc/self/maps')
	enable_binreloc = true
  else
	error('/proc/self/maps not available. Binary relocation cannot be enabled')
  endif
elif binreloc.auto()
  prefix = get_option('prefix')
  everything_in_one_place = (['/usr', '/usr/local'].contains(prefix)) or \
	  (get_option('bindir').startswith(prefix) and get_option('sbindir').startswith(prefix) and \
	  get_option('datadir').startswith(prefix) and get_option('libdir').startswith(prefix) and \
	  get_option('libexecdir').startwith(prefix) and get_option('sysconfdir').startwith(prefix))
  enable_binreloc = fs.exists('/proc/self/maps') and everything_in_one_place
endif

if enable_binreloc
  conf.set('ENABLE_BINRELOC')
endif

summary({'Enable binary relocation', enable_binreloc.to_string('yes', 'no')})

# CTags source compatibility (we actually use GRegex instead of POSIX regcomp)

conf.set('HAVE_REGCOMP', 1)

# Plugins support

if not get_option('disable-plugins')
  conf.set('HAVE_PLUGINS', 1)
  summary({'Build with plugin support': 'yes'})
else
  summary({'Build with plugin support': 'no'})
endif

# check for mingw specific settings

use_socket = get_option('enable-socket')
use_vte = get_option('enable-vte') 

if host_machine.system() == "windows"
  conf.set('WIN32', 1)
  use_socket = true
  use_vte = false
  pkgdatadir = get_option('prefix') / 'data'
else
  pkgdatadir = get_option('datadir') / 'geany'
endif

if use_vte
  conf.set('HAVE_VTE', 1)
endif

if use_socket
  conf.set('HAVE_SOCKET', 1)
endif # TODO Solaris libsocket

libsocket = dependency('socket', required: false)

summary({'Use UNIX domain socket support': use_socket.to_string('yes', 'no'),
		'Use virtual terminal support (VTE)': use_vte.to_string('yes', 'no')})

if get_option('enable-mac-integration')
  if use_gtk2
	gtk_mac_integration = dependency('gtk-mac-integration-gtk2')
  else
	gtk_mac_integration = dependency('gtk-mac-integration-gtk3')
  endif
else
  gtk_mac_integration = disabler()
endif


# GEANY_CHECK_THE_FORCE dnl hehe

# i18n

subdir('po')

# The default Python command.  On Windows, use the `py` launcher by default

python_command = get_option('python-command')
if python_command == 'auto'
  if host_machine.system() == 'windows'
	python_command = 'py'
  elif
	python_command = 'python'
  endif
endif

# Documentation tools

# I don't think it is correct to store prebuilt files in tree

have_prebuilt_html_files = fs.is_file(meson.project_source_root() / 'doc/geany.html')

html_docs_default = (revision == '-1' or have_prebuilt_html_files) ? 'auto' : 'yes'

if get_option('html-docs').enabled()
  geany_enable_html_docs = 'yes'
elif get_option('html-docs').disabled()
  geany_enable_html_docs = 'no'
else
  geany_enable_html_docs = html_docs_default

if geany_enable_html_docs != 'no'
  rst2html = find_program('rst2html', required: false)
  if rst2html.found()
	geany_enable_html_docs = 'yes'
  elif geany_enable_html_docs == 'yes'
	error('''Documentation enabled but rst2html not found.
You can explicitly disable building of the HTML manual with -Dhtml-docs=disabled,
but you then may not have a local copy of the HTML manual.''')
  else
	geany_enable_html_docs = 'no'
  endif
endif

enable_pdf_docs = get_option('pdf-docs')
rst2pdf = find_program('rst2pdf', required: enable_pdf_docs)

enable_api_docs = get_option('api-docs')
doxygen = find_program('doxygen', required: enable_api_docs)

gtkdoc_header = get_option('gtkdoc-header')

if gtkdoc_header.enabled() and not doxygen.found()
  error('GtkDoc enabled but Doxygen not available')

summary({'Build HTML documentation': geany_enable_html_docs,
		'Build PDF documentation': enable_pdf_docs.enabled().to_string('yes', 'no'),
		'Build API documentation': enable_api_docs.enabled().to_string('yes', 'no'),
		'Generate GtkDoc header': gtkdoc_header.enabled().to_string('yes', 'no')})

python2 = pymod.find_installation('python2', modules: 'lxml', required: gtkdoc_header)

# libgeany 

if host_machine.system() == 'windows'
  libgeany_export_cflags = ['-DGEANY_EXPORT_SYMBOL="__declspec(dllexport)"']
elif cc.has_function_attribute('visibility')
  libgeany_export_cflags = ['-fvisibility=hidden', '-DGEANY_EXPORT_SYMBOL="__attribute((visibility(\"default\")))']
endif

libgeany_export_cflags += '-DGEANY_API_SYMBOL=GEANY_EXPORT_SYMBOL'

libgeany_cflags = libgeany_export_cflags
libgeany_export_ldflags = ['-version-info 0:0:0'] # TODO

sort = find_program('sort', required: true)
uniq = find_program('uniq', required: true)

configure_file(output: 'config.h', configuration: conf)

foreach dir: ['ctags', 'scintilla', 'src', 'plugins', 'icons', 'doc', 'data', 'tests']
  subdir(dir)
endforeach


# AC_CONFIG_FILES([
# 		Makefile
# 		icons/Makefile
# 		icons/16x16/Makefile
# 		icons/24x24/Makefile
# 		icons/32x32/Makefile
# 		icons/48x48/Makefile
# 		icons/scalable/Makefile
# 		icons/tango/Makefile
# 		icons/tango/16x16/Makefile
# 		icons/tango/24x24/Makefile
# 		icons/tango/32x32/Makefile
# 		icons/tango/48x48/Makefile
# 		icons/tango/scalable/Makefile
# 		ctags/Makefile
# 		scintilla/Makefile
# 		scintilla/include/Makefile
# 		src/Makefile
# 		src/tagmanager/Makefile
# 		plugins/Makefile
# 		po/Makefile.in
# 		data/Makefile
# 		data/filedefs/filetypes.python
# 		doc/Makefile
# 		doc/geany.1
# 		geany.pc
# 		geany.nsi
# 		doc/Doxyfile
# 		tests/Makefile
# 		tests/ctags/Makefile
# ])


### Process this file with automake to produce Makefile.in
##
##SUBDIRS = ctags scintilla src plugins icons po doc data tests
##
##AUTOMAKE_OPTIONS =	1.7
##
##ACLOCAL_AMFLAGS = -I m4
##AM_DISTCHECK_CONFIGURE_FLAGS = --enable-api-docs --enable-html-docs --enable-pdf-docs \
##                              --enable-gtkdoc-header
##
##WIN32_BUILD_FILES = \
##	geany_private.rc \
##	geany.exe.manifest
##
##EXTRA_DIST = \
##	autogen.sh \
##	scripts/gen-api-gtkdoc.py \
##	geany.desktop.in \
##	geany.pc.in \
##	ChangeLog.pre-1-22 \
##	HACKING \
##	README.I18N \
##	README.Packagers \
##	intltool-extract.in \
##	intltool-merge.in \
##	intltool-update.in \
##	$(WIN32_BUILD_FILES)
##
##DISTCLEANFILES = \
##	geany.desktop \
##	intltool-extract \
##	intltool-merge \
##	intltool-update
##
##uninstall-local:
##	rm -rf $(DESTDIR)$(pkgdatadir);
##
#  manually install some files under another name
# install-data-local:
# 	$(mkinstalldirs) $(DESTDIR)$(pkgdatadir)
# 	$(INSTALL_DATA) $(srcdir)/COPYING $(DESTDIR)$(pkgdatadir)/GPL-2
# if MINGW
# 	$(INSTALL_DATA) $(srcdir)/AUTHORS $(DESTDIR)$(prefix)/Authors.txt
# 	$(INSTALL_DATA) $(srcdir)/ChangeLog $(DESTDIR)$(prefix)/Changelog.txt
# 	$(INSTALL_DATA) $(srcdir)/COPYING $(DESTDIR)$(prefix)/Copying.txt
# 	$(INSTALL_DATA) $(srcdir)/README $(DESTDIR)$(prefix)/Readme.txt
# 	$(INSTALL_DATA) $(srcdir)/NEWS $(DESTDIR)$(prefix)/News.txt
# 	$(INSTALL_DATA) $(srcdir)/THANKS $(DESTDIR)$(prefix)/Thanks.txt
# 	$(INSTALL_DATA) $(srcdir)/TODO $(DESTDIR)$(prefix)/Todo.txt
# endif
# 
# dist-hook:
# 	@if test -d "$(top_srcdir)/.git"; then \
# 		echo '  GEN    ChangeLog'; \
# 		( cd "$(top_srcdir)" && \
# 		  echo '# Generated by Makefile. Do not edit.' && echo && \
# 		  GIT_CONFIG_NOSYSTEM=1 HOME="$(srcdir)" XDG_CONFIG_HOME="$(srcdir)" \
# 		  git log --stat 0.21.0.. ) > ChangeLog.tmp \
# 		&& mv -f ChangeLog.tmp "$(distdir)/ChangeLog" \
# 		|| ( rm -f ChangeLog.tmp ; \
# 		     echo 'Failed to generate ChangeLog' >&2 ); \
# 	else \
# 		echo 'A git clone is required to generate a ChangeLog' >&2; \
# 	fi
# 
# 
# sign:
# 	if test -f $(PACKAGE)-$(VERSION).tar.gz; then \
# 		gpg --detach-sign --digest-algo SHA512 $(PACKAGE)-$(VERSION).tar.gz; fi
# 	if test -f $(PACKAGE)-$(VERSION).tar.bz2; then \
# 		gpg --detach-sign --digest-algo SHA512 $(PACKAGE)-$(VERSION).tar.bz2; fi
# 
# 
# rpm: dist
# 	rpmbuild -ta $(distdir).tar.gz
# 
# 
# pkgconfig_DATA = geany.pc
# pkgconfigdir = $(libdir)/pkgconfig
# 
# desktopdir = $(datadir)/applications
# desktop_in_files = geany.desktop
# desktop_DATA = $(desktop_in_files:.desktop.in=.desktop)
# @INTLTOOL_DESKTOP_RULE@



