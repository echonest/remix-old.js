REMIX_SRC_FILES = FileList.new('src/js/**/*.js')
EXAMPLE_STATIC_FILES = FileList.new('src/editor/*') { |fl| fl.exclude('src/editor/player.mxml') }

LIBRARIES = ['build/echo-nest-flash-api.swc', 'build/flash-audio.swc', 'lib/as3corelib-.92.1/lib/as3corelib.swc']

STATIC_FILES = ['README.markdown', 'LICENSE.txt']

task :init => ['lib/MP3FileReferenceLoaderLib/MP3FileReferenceLoaderLib.swc', 'lib/as3corelib-.92.1/lib/as3corelib.swc', :submodule]

task :submodule do
  sh 'git submodule init'
  sh 'git submodule update'
end

file 'lib/flash-audio/src' do
  Rake::Task[:submodule].invoke
end

file 'lib/echo-nest-flash-api/src' do
  Rake::Task[:submodule].invoke
end

file 'lib/as3corelib-.92.1/lib/as3corelib.swc' do
  sh 'curl -O http://as3corelib.googlecode.com/files/as3corelib-.92.1.zip && unzip as3corelib-.92.1.zip as3corelib-.92.1/lib/as3corelib.swc -d lib && rm as3corelib-.92.1.zip'
end

file 'lib/MP3FileReferenceLoaderLib' do
  sh 'svn co http://echo-nest-remix.googlecode.com/svn/trunk/flash/MP3FileReferenceLoaderLib lib/MP3FileReferenceLoaderLib'
end

file 'lib/MP3FileReferenceLoaderLib/MP3FileReferenceLoaderLib.swc' => ['lib/MP3FileReferenceLoaderLib'] do
  sh 'compc -source-path lib/MP3FileReferenceLoaderLib/src -include-classes org/audiofx/mp3/ByteArraySegment org/audiofx/mp3/MP3FileReferenceLoader org/audiofx/mp3/MP3Parser org/audiofx/mp3/MP3SoundEvent org/audiofx/mp3/SoundClassSwfByteCode -o lib/MP3FileReferenceLoaderLib/MP3FileReferenceLoaderLib.swc'
end

directory 'build'

def swc(o, path, swc_deps=[], &block)
  as_files = FileList["#{path}/**/*.as"]
  library_path = swc_deps.join(',')
  file o => as_files + swc_deps + [path, 'build'] do
    yield if block_given?
    # do this again if there weren't any files when we created the list
    as_files = FileList["#{path}/**/*.as"] if as_files.length == 0
    path_len = path.length + 1
    include_classes = as_files.to_a.map {|f| f[path_len...-3]}.join(' ')
    sh "compc -source-path #{path} -include-classes #{include_classes} -library-path+=#{library_path} -o #{o}"
  end
end

swc 'build/flash-audio.swc', 'lib/flash-audio/src'
swc 'build/echo-nest-flash-api.swc', 'lib/echo-nest-flash-api/src', ['lib/MP3FileReferenceLoaderLib/MP3FileReferenceLoaderLib.swc', 'lib/as3corelib-.92.1/lib/as3corelib.swc']

directory 'dist'

file 'dist/remix.swf' => LIBRARIES do
  sh "mxmlc -library-path+=#{LIBRARIES.join(',')} -output dist/remix.swf -- src/editor/player.mxml"
end

file 'dist/remix.js' => ['dist'] + REMIX_SRC_FILES do
  sh "cat #{REMIX_SRC_FILES} > dist/remix.js"
end

directory 'dist/example'

file 'dist/example' => ['dist/remix.swf', 'dist/remix.js'] + EXAMPLE_STATIC_FILES do
  sh "cp #{EXAMPLE_STATIC_FILES} dist/example"
  sh 'cp dist/remix.swf dist/example'
end

task :example => ['dist/example']

task :swf => ['dist/remix.swf']

task :js => ['dist/remix.js']

task :clean do
  sh 'rm -rf dist'
  sh 'rm -rf build'
end

directory 'dist/src'
file 'dist/src' => LIBRARIES do
  sh 'mkdir -p dist/src/lib'
  sh "cp #{LIBRARIES.join(' ')} dist/src/lib"
  sh 'cp -r src dist/src/src'
  sh "cp #{STATIC_FILES.join(' ')} dist/src"
end

task :src_dist => 'dist/src'