##SUBDIRS = include

subdir('include')

#
#AM_CXXFLAGS = -DNDEBUG -DGTK -DSCI_LEXER -DNO_CXX11_REGEX
#

#marshallers: gtk/scintilla-marshal.list
#	glib-genmarshal --prefix scintilla_marshal gtk/scintilla-marshal.list --header > gtk/scintilla-marshal.h
#	glib-genmarshal --prefix scintilla_marshal gtk/scintilla-marshal.list --body > gtk/scintilla-marshal.c

gnome = import('gnome')

scintilla_marshalled = gnome.genmarshal('gtk/scintilla-marshal',
										prefix: 'scintilla_marshal',
										sources: 'gtk/scintilla-marshal.list'
									  )

lexer_sources = [
  'lexers/LexAbaqus.cxx',
  'lexers/LexAda.cxx',
  'lexers/LexAsm.cxx',
  'lexers/LexBash.cxx',
  'lexers/LexBasic.cxx',
  'lexers/LexBatch.cxx',
  'lexers/LexCOBOL.cxx',
  'lexers/LexCPP.cxx',
  'lexers/LexCSS.cxx',
  'lexers/LexCaml.cxx',
  'lexers/LexCmake.cxx',
  'lexers/LexCoffeeScript.cxx',
  'lexers/LexD.cxx',
  'lexers/LexDiff.cxx',
  'lexers/LexErlang.cxx',
  'lexers/LexForth.cxx',
  'lexers/LexFortran.cxx',
  'lexers/LexHTML.cxx',
  'lexers/LexHaskell.cxx',
  'lexers/LexLaTeX.cxx',
  'lexers/LexLisp.cxx',
  'lexers/LexLua.cxx',
  'lexers/LexMake.cxx',
  'lexers/LexMarkdown.cxx',
  'lexers/LexMatlab.cxx',
  'lexers/LexNsis.cxx',
  'lexers/LexNull.cxx',
  'lexers/LexPascal.cxx',
  'lexers/LexPerl.cxx',
  'lexers/LexPowerShell.cxx',
  'lexers/LexProps.cxx',
  'lexers/LexPython.cxx',
  'lexers/LexPO.cxx',
  'lexers/LexR.cxx',
  'lexers/LexRuby.cxx',
  'lexers/LexRust.cxx',
  'lexers/LexSmalltalk.cxx',
  'lexers/LexSQL.cxx',
  'lexers/LexTCL.cxx',
  'lexers/LexTxt2tags.cxx',
  'lexers/LexVHDL.cxx',
  'lexers/LexVerilog.cxx',
  'lexers/LexYAML.cxx'
]

sources = [
  'gtk/PlatGTK.cxx',
  'gtk/ScintillaGTK.cxx',
  'gtk/ScintillaGTKAccessible.cxx',
  'gtk/scintilla-marshal.c',
  'lexlib/Accessor.cxx',
  'lexlib/CharacterCategory.cxx',
  'lexlib/CharacterSet.cxx',
  'lexlib/DefaultLexer.cxx',
  'lexlib/LexerBase.cxx',
  'lexlib/LexerModule.cxx',
  'lexlib/LexerNoExceptions.cxx',
  'lexlib/LexerSimple.cxx',
  'lexlib/PropSetSimple.cxx',
  'lexlib/StyleContext.cxx',
  'lexlib/WordList.cxx',
  'src/AutoComplete.cxx',
  'src/CallTip.cxx',
  'src/CaseConvert.cxx',
  'src/CaseFolder.cxx',
  'src/Catalogue.cxx',
  'src/CellBuffer.cxx',
  'src/CharClassify.cxx',
  'src/ContractionState.cxx',
  'src/ContractionState.h',
  'src/DBCS.cxx',
  'src/Decoration.cxx',
  'src/Document.cxx',
  'src/Editor.cxx',
  'src/EditModel.cxx',
  'src/EditView.cxx',
  'src/ExternalLexer.cxx',
  'src/Indicator.cxx',
  'src/IntegerRectangle.h',
  'src/KeyMap.cxx',
  'src/LineMarker.cxx',
  'src/MarginView.cxx',
  'src/PerLine.cxx',
  'src/PositionCache.cxx',
  'src/RESearch.cxx',
  'src/RunStyles.cxx',
  'src/ScintillaBase.cxx',
  'src/Selection.cxx',
  'src/Style.cxx',
  'src/UniConversion.cxx',
  'src/UniqueString.cxx',
  'src/ViewStyle.cxx',
  'src/XPM.cxx',
]

#$(LEXER_SRCS)
#
#libscintilla_la_SOURCES = $(SRCS)
#
#AM_CPPFLAGS = -I$(top_srcdir) -I$(srcdir)/include -I$(srcdir)/src -I$(srcdir)/lexlib \
#	@GTK_CFLAGS@ @LIBGEANY_CFLAGS@
#
#
#EXTRA_DIST=gtk/scintilla-marshal.list License.txt README version.txt
