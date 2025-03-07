project(
  'evince', ['c'],
  version: '43.alpha',
  license: 'GPL-2.0-or-later',
  meson_version: '>= 0.53.0',
  default_options: [
    'buildtype=debugoptimized',
    'libhandy:vapi=false',
    'libhandy:introspection=disabled',
    'libhandy:tests=false',
  ]
)

if get_option('development')
  app_id = 'org.gnome.Evince.Devel'
else
  app_id = 'org.gnome.Evince'
endif

ev_name = meson.project_name().to_lower()

ev_version = meson.project_version()
version_array = ev_version.split('.')
ev_major_version = version_array[0].to_int()
ev_minor_version = version_array[1]

ev_prefix = get_option('prefix')
ev_bindir = join_paths(ev_prefix, get_option('bindir'))
ev_datadir = join_paths(ev_prefix, get_option('datadir'))
ev_includedir = join_paths(ev_prefix, get_option('includedir'))
ev_libdir = join_paths(ev_prefix, get_option('libdir'))
ev_libexecdir = join_paths(ev_prefix, get_option('libexecdir'))
ev_localedir = join_paths(ev_prefix, get_option('localedir'))
ev_mandir = join_paths(ev_prefix, get_option('mandir'))

ev_pkgdatadir = join_paths(ev_datadir, ev_name)

ev_appstreamdir = join_paths(ev_datadir, 'metainfo')

# Libtool versioning. The backend and view libraries have separate versions.
# Before making a release, the libtool version should be modified.
# The string is of the form C:R:A.
# - If interfaces have been changed or added, but binary compatibility has
#   been preserved, change to C+1:0:A+1
# - If binary compatibility has been broken (eg removed or changed interfaces)
#   change to C+1:0:0
# - If the interface is the same as the previous version, change to C:R+1:A

# Libtool version of the backend library
ev_document_current = 4
ev_document_revision = 0
ev_document_age = 0
ev_document_version = '@0@.@1@.@2@'.format(ev_document_current, ev_document_revision, ev_document_age)
ev_document_current_minus_age = ev_document_current - ev_document_age

# Libtool version of the view library
ev_view_current = 3
ev_view_revision = 0
ev_view_age = 0
ev_view_version = '@0@.@1@.@2@'.format(ev_view_current, ev_view_revision, ev_view_age)
ev_view_current_minus_age = ev_view_current - ev_view_age

ev_api_version = '3.0'

ev_include_subdir = join_paths(ev_name, ev_api_version)

# Backends directory
ev_binary_version = ev_document_current
ev_backends_binary_version = ev_binary_version
ev_backends_subdir = join_paths(ev_name, ev_backends_binary_version.to_string(), 'backends')
ev_backendsdir = join_paths(ev_libdir, ev_backends_subdir)

ev_namespace = 'org.gnome.Evince'

ev_code_prefix = 'Ev'

ev_debug = get_option('buildtype').contains('debug')

cc = meson.get_compiler('c')

config_h = configuration_data()
config_h.set10('_GNU_SOURCE', true)

# package
config_h.set_quoted('PACKAGE_ICON_NAME', app_id)
config_h.set_quoted('PACKAGE_VERSION', ev_version)
config_h.set_quoted('VERSION', ev_version)

# i18n
config_h.set_quoted('GETTEXT_PACKAGE', ev_name)
config_h.set('ENABLE_NLS', true)

# Support for nl_langinfo (_NL_MEASUREMENT_MEASUREMENT) (optional)
langinfo_measurement_src = '''
  #include <langinfo.h>
  int main() {
    char c;
    c = *((unsigned char *)  nl_langinfo(_NL_MEASUREMENT_MEASUREMENT));
  };
'''
config_h.set('HAVE__NL_MEASUREMENT_MEASUREMENT', cc.compiles(langinfo_measurement_src, name: 'Support for nl_langinfo'),
             description: 'Define if _NL_MEASUREMENT_MEASUREMENT is available')

# compiler flags
common_flags = ['-DHAVE_CONFIG_H'] + cc.get_supported_arguments([
  '-Wno-deprecated-declarations',
])

common_ldflags = []

if build_machine.system() == 'windows'
  common_flags += '-D_WIN32_WINNT=0x0500'

  common_ldflags = cc.get_supported_link_arguments('-mwindows')
endif

# GLib on macOS expects so as shared_module suffix, while meson uses dylib by default
if host_machine.system() == 'darwin'
  name_suffix = 'so'
else
  name_suffix = []
endif

if ev_debug
  common_flags += ['-DEV_ENABLE_DEBUG'] + cc.get_supported_arguments([
    '-Wnested-externs',
    '-Wstrict-prototypes',
    '-Werror=format=2',
    '-Werror=implicit-function-declaration',
    '-Werror=init-self',
    '-Werror=missing-include-dirs',
    '-Werror=missing-prototypes',
    '-Werror=pointer-arith',
    '-Werror=return-type',
  ])
endif

add_project_arguments(common_flags, language: 'c')

gnome = import('gnome')
i18n = import('i18n')
pkg = import('pkgconfig')

source_root = meson.current_source_dir()

data_dir = join_paths(source_root, 'data')
po_dir = join_paths(source_root, 'po')

top_inc = include_directories('.')

glib_req_version = '>= 2.44.0'
gtk_req_version = '>= 3.22.0'
hdy_req_version = '>= 1.5.0'

gdk_pixbuf_dep = dependency('gdk-pixbuf-2.0', version: '>= 2.40.0')
config_h.set_quoted('EXTRA_GDK_PIXBUF_LOADERS_DIR',
                    join_paths (ev_libdir, ev_name, 'gdk-pixbuf', gdk_pixbuf_dep.get_pkgconfig_variable('gdk_pixbuf_binary_version')))

gio_dep = dependency('gio-2.0', version: glib_req_version)
glib_dep = dependency('glib-2.0', version: glib_req_version)
gmodule_dep = dependency('gmodule-2.0')
gmodule_no_export_dep = dependency('gmodule-no-export-2.0', version: glib_req_version)
gtk_dep = dependency('gtk+-3.0', version: gtk_req_version)
gthread_dep = dependency('gthread-2.0', version: glib_req_version)
# Keep the version here synchronised with subprojects/libhandy.wrap
hdy_dep = dependency('libhandy-1', version: hdy_req_version, fallback: ['libhandy', 'libhandy_dep'])

m_dep = cc.find_library('m')

gidocgen_dep = dependency('gi-docgen', version: '>= 2021.1',
			  fallback: ['gi-docgen', 'dummy_dep'],
			  required: get_option('gtk_doc'))

# Although GTK+ 3.10 includes hi-dpi functionality, it does not require a cairo with
# cairo_surface_set_device_scale(), which we also need if we're to support hi-dpi,
# so we need check for that explicitly.
cairo_dep = dependency('cairo', version: '>= 1.10.0')
config_h.set('HAVE_HIDPI_SUPPORT', cc.has_function('cairo_surface_set_device_scale', dependencies: cairo_dep))

# ZLIB support (required)
zlib_dep = cc.find_library('z', required: false)
assert(zlib_dep.found() and cc.has_function('inflate', dependencies: zlib_dep) and cc.has_function('crc32', dependencies: zlib_dep),
      'No sufficient zlib library found on your system')

ev_platform = get_option('platform')
if ev_platform == 'gnome'
  # Evince has a rather soft run-time dependency on hicolor-icon-theme.
  # If the hicolor theme is not available, Evince fails to display some
  # icons. Because we cannot check for it at run-time, we instead
  # would like to require the icon theme at compile-time. But, because
  # the hicolor-icon-theme does not have a pkgconfig file, on gnome we
  # require the gnome icon theme instead.
  adwaita_icon_theme_dep = dependency('adwaita-icon-theme', version: '>= 2.17.1')

  # *** Nautilus property page build ***
  enable_nautilus = get_option('nautilus')
  if enable_nautilus
    libnautilus_extension_dep = dependency('libnautilus-extension', version: '>= 3.28.0')
    nautilus_extension_dir = libnautilus_extension_dep.get_pkgconfig_variable('extensiondir', define_variable: ['libdir', ev_libdir])
  endif

  # *** DBUS ***
  enable_dbus = get_option('dbus')
  if enable_dbus
    # Check for dbus service dir
    dbus_service_dir = dependency('dbus-1').get_pkgconfig_variable('session_bus_services_dir', define_variable: ['datadir', ev_datadir])
  endif
  config_h.set('ENABLE_DBUS', enable_dbus)

  # *** GNOME Keyring support ***
  libsecret_dep = dependency('libsecret-1', version: '>= 0.5', required: get_option('keyring'))
  enable_keyring = libsecret_dep.found()
  config_h.set('WITH_KEYRING', enable_keyring)

  # GKT+ Unix Printing
  gtk_unix_print_dep = dependency('gtk+-unix-print-3.0', version: gtk_req_version, required: get_option('gtk_unix_print'))
  enable_gtk_unix_print = gtk_unix_print_dep.found()
  config_h.set10('GTKUNIXPRINT_ENABLED', enable_gtk_unix_print)

  if enable_dbus or enable_gtk_unix_print
    gio_unix_dep = dependency('gio-unix-2.0', version: glib_req_version)
  else
    gio_unix_dep = dependency('', required: false)
  endif

else
  message('Evince has a soft run-time dependency on hicolor-icon-theme. You are advised to have this theme installed when running Evince.')
  enable_nautilus = false
  enable_dbus = false
  enable_keyring = false
  enable_gtk_unix_print = false
  gio_unix_dep = dependency('', required: false)
  gtk_unix_print_dep = dependency('', required: false)
endif

# *** GObject Introspection ***
enable_introspection = get_option('introspection')
if enable_introspection
  dependency('gobject-introspection-1.0', version: '>= 1.0')
endif

# *** GNOME Desktop (Thumbnail cache) ***
gnome_desktop_dep = dependency('gnome-desktop-3.0', required: get_option('thumbnail_cache'))
enable_thumbnail_cache = gdk_pixbuf_dep.found() and gnome_desktop_dep.found()
config_h.set('HAVE_LIBGNOME_DESKTOP', enable_thumbnail_cache)

# *** GStreamer (Multimedia) ***
gstreamer_base_dep = dependency('gstreamer-base-1.0', required: get_option('multimedia'))
gstreamer_dep = dependency('gstreamer-1.0', required: get_option('multimedia'))
gstreamer_video_dep = dependency('gstreamer-video-1.0', required: get_option('multimedia'))

enable_multimedia = gstreamer_dep.found() and gstreamer_base_dep.found() and gstreamer_video_dep.found()
config_h.set('ENABLE_MULTIMEDIA', enable_multimedia)

# *** Gspell ***
gspell_dep = dependency('gspell-1', version: '>= 1.6.0', required: get_option('gspell'))
enable_gspell = gspell_dep.found()
config_h.set10('WITH_GSPELL', enable_gspell)

# *** systemd user unit dir ***
systemd_user_unit_dir = get_option('systemduserunitdir')
install_systemd_user_unit_dir = (systemd_user_unit_dir != 'no')

if install_systemd_user_unit_dir and systemd_user_unit_dir == ''
  systemd_user_unit_dir = join_paths(ev_prefix, 'lib', 'systemd', 'user')
endif

# *** Check for Desktop Schemas ***
gsettings_desktop_schemas_dep = dependency('gsettings-desktop-schemas', required: false)
config_h.set('HAVE_DESKTOP_SCHEMAS', gsettings_desktop_schemas_dep.found())

# *** libsynctex ***
if get_option('internal_synctex') == 'true'
  external_synctex = false
else
  synctex_dep = dependency('synctex', version: '>= 1.19', required: false)
  external_synctex = synctex_dep.found()

  if not external_synctex and get_option('internal_synctex') == 'false'
    error('External synctex requested but not found')
  endif
endif

# *** Mime types list ***
mime_types_list = {
  'comics': [
    'application/vnd.comicbook-rar',
    'application/vnd.comicbook+zip',
    'application/x-cb7',
    'application/x-cbr',
    'application/x-cbt',
    'application/x-cbz',
    'application/x-ext-cb7',
    'application/x-ext-cbr',
    'application/x-ext-cbt',
    'application/x-ext-cbz',
  ],
  'djvu': [
    'application/x-ext-djv',
    'application/x-ext-djvu',
    'image/vnd.djvu',
  ],
  'dvi': [
    'application/x-bzdvi',
    'application/x-dvi',
    'application/x-ext-dvi',
    'application/x-gzdvi',
  ],
  'illustrator': [
    'application/illustrator'
  ],
  'pdf': [
    'application/pdf',
    'application/x-bzpdf',
    'application/x-ext-pdf',
    'application/x-gzpdf',
    'application/x-xzpdf',
  ],
  'ps': [
    'application/postscript',
    'application/x-bzpostscript',
    'application/x-gzpostscript',
    'application/x-ext-eps',
    'application/x-ext-ps',
    'image/x-bzeps',
    'image/x-eps',
    'image/x-gzeps',
  ],
  'tiff': [
    'image/tiff'
  ],
  'xps': [
    'application/oxps',
    'application/vnd.ms-xpsdocument',
  ],
}

backends = {}
evince_mime_types = []

# *** Spectre ***
if not get_option('ps').disabled() or not get_option('dvi').disabled()
  # libspectre (used by ps and dvi backends)
  libspectre_req_version = '>= 0.2.0'
  libspectre_dep = dependency('libspectre', version: libspectre_req_version, required: false)
  config_h.set('HAVE_SPECTRE', libspectre_dep.found())
else
  libspectre_dep = disabler()
endif

# *** Comic Book ***
libarchive_req_version = '>= 3.6.0'
libarchive_dep = dependency('libarchive', version: libarchive_req_version, required: get_option('comics'))
enable_comics = libarchive_dep.found()
if enable_comics
  backends += {'comics': mime_types_list.get('comics')}
  evince_mime_types += mime_types_list.get('comics')
elif get_option('comics').auto()
  warning('** Comics support is disabled since libarchive (version ' + libarchive_req_version + ') is needed')
endif

# *** DJVU ***
ddjvuapi_req_version = '>= 3.5.22'
ddjvuapi_dep = dependency('ddjvuapi', version: ddjvuapi_req_version, required: get_option('djvu'))
enable_djvu = ddjvuapi_dep.found()
if enable_djvu
  backends += {'djvu': mime_types_list.get('djvu')}
  evince_mime_types += mime_types_list.get('djvu')
elif get_option('djvu').auto()
  warning('Djvu support is disabled since a recent version of the djvulibre library was not found. You need at least djvulibre ' + ddjvuapi_req_version + ' which can be found on http://djvulibre.djvuzone.org')
endif

# *** DVI ***
kpathsea_dep = cc.find_library('kpathsea', required: get_option('dvi'))
enable_dvi = kpathsea_dep.found() and cc.has_function('kpse_init_prog', dependencies: kpathsea_dep)
if enable_dvi
  config_h.set10('STDC_HEADERS', true)

  if not cc.has_type('size_t', prefix: '#include<sys/types.h>')
    config_h.set('size_t', 'unsigned int')
  endif

  types = [
    ['short', 'SHORT'],
    ['int', 'INT'],
    ['long', 'LONG'],
    ['long long', 'LONG_LONG'],
    ['void  *', 'VOID_P'],
  ]

  foreach type: types
    config_h.set('SIZEOF_' + type[1], cc.sizeof(type[0]))
  endforeach

  backends += {'dvi': mime_types_list.get('dvi')}
  evince_mime_types += mime_types_list.get('dvi')
elif get_option('dvi').auto()
  warning('Dvi support is disabled since kpathsea library was not found. Check your installation.')
endif

# *** PDF ***
poppler_req_version = '>= 22.02.0'
poppler_glib_dep = dependency('poppler-glib', version: poppler_req_version, required: get_option('pdf'))

libxml_req_version = '>= 2.5.0'
libxml_dep = dependency('libxml-2.0', version: libxml_req_version, required: get_option('pdf'))

enable_pdf = poppler_glib_dep.found() and libxml_dep.found()
if enable_pdf
  cairo_pdf_dep = dependency('cairo-pdf', required: false)
  cairo_ps_dep = dependency('cairo-ps', required: false)

  backends += {'pdf': mime_types_list.get('pdf')}
  evince_mime_types += mime_types_list.get('pdf')
elif get_option('pdf').auto()
  warning('PDF support is disabled since poppler-glib version ' + poppler_req_version + ' or libxml-2.0 version ' + libxml_req_version + ' not found')
endif

# *** PostScript ***
enable_ps = not get_option('ps').disabled() and libspectre_dep.found()
if enable_ps
  backends += {'ps': mime_types_list.get('ps')}
  evince_mime_types += mime_types_list.get('ps')
elif not get_option('ps').disabled()
  str = 'PS support is disabled since libspectre (version ' + libspectre_req_version + ') is needed'
  if get_option('ps').auto()
    error(str)
  endif
  warning(str)
endif

# *** TIFF ***
libtiff_dep = dependency('libtiff-4', required: get_option('tiff'))
enable_tiff = libtiff_dep.found()
if enable_tiff
  backends += {'tiff': mime_types_list.get('tiff')}
  evince_mime_types += mime_types_list.get('tiff')
elif get_option('tiff').auto()
  warning('Tiff support is disabled since tiff library version 4.0 or newer not found')
endif

# *** XPS ***
libgxps_req_version = '>= 0.2.1'
libgxps_dep = dependency('libgxps', version: libgxps_req_version, required: get_option('xps'))
enable_xps = libgxps_dep.found()
if enable_xps
  backends += {'xps': mime_types_list.get('xps')}
  evince_mime_types += mime_types_list.get('xps')
elif get_option('xps').auto()
  warning('** XPS support is disabled since libgxps (version ' + libgxps_req_version + ') is needed')
endif

if enable_pdf and enable_ps
  backends += {
    'pdf': mime_types_list.get('pdf') + mime_types_list.get('illustrator'),
    'ps': mime_types_list.get('ps') + mime_types_list.get('illustrator'),
  }
  evince_mime_types += mime_types_list.get('illustrator')
endif

mime_types_conf = configuration_data()
mime_types_conf.set('EVINCE_MIME_TYPES', ';'.join(evince_mime_types))
mime_types_conf.set('PACKAGE_ICON_NAME', app_id)

subdir('cut-n-paste')
subdir('libdocument')
subdir('backend')
subdir('libview')
subdir('libmisc')
subdir('properties')

# *** Document Viewer ***
enable_viewer = get_option('viewer')
if enable_viewer
  subdir('shell')
endif

subdir('po')
subdir('help')

# *** Thumbnailer ***
enable_thumbnailer = get_option('thumbnailer')
if enable_thumbnailer
  subdir('thumbnailer')
endif

# Print Previewer
enable_previewer = get_option('previewer')
if enable_previewer
  subdir('previewer')
endif

subdir('data')

headers = files(
  'evince-document.h',
  'evince-view.h',
)

install_headers(
  headers,
  subdir: ev_include_subdir,
)

# Appdata files
appdata = ev_namespace + '.appdata.xml'

i18n.merge_file(
  input: appdata + '.in',
  output: appdata,
  po_dir: po_dir,
  install: true,
  install_dir: ev_appstreamdir,
)

configure_file(
  output: 'config.h',
  configuration: config_h,
)

meson.add_install_script(
  'meson_post_install.py',
  ev_datadir,
)


summary({'Platform...................': ev_platform,
         'Debug mode.................': ev_debug,
        }, section: 'General', bool_yn: true)
summary({'Viewer.....................': enable_viewer,
         'Previewer..................': enable_previewer,
         'Thumbnailer................': enable_thumbnailer,
         'Nautilus extension.........': enable_nautilus,
        }, section: 'Frontends', bool_yn: true)
summary({'Comics.....................': enable_comics,
         'DJVU.......................': enable_djvu,
         'DVI........................': enable_dvi,
         'PDF........................': enable_pdf,
         'PostScript.................': enable_ps,
         'TIFF.......................': enable_tiff,
         'XPS........................': enable_xps,
        }, section: 'Backends', bool_yn: true)
summary({'Gtk-doc reference..........': enable_gtk_doc,
         'User documentation.........': enable_user_doc,
         'GObject introspection......': enable_introspection,
         'DBus communication.........': enable_dbus,
         'Systemd units installation.': systemd_user_unit_dir,
         'Keyring integration........': enable_keyring,
         'GTK+ Unix print ...........': enable_gtk_unix_print,
         'Thumbnail cache ...........': enable_thumbnail_cache,
         'Multimedia ................': enable_multimedia,
         'Spell checker .............': enable_gspell,
         'SyncTex ...................': external_synctex.to_string('external', 'internal'),
        }, section: 'Features', bool_yn: true)
