fs   = require 'fs'
exec = require('child_process').exec

# ANSI Terminal Colors.
bold  = '\033[0;1m'
red   = '\033[0;31m'
green = '\033[0;32m'
reset = '\033[0m'

# Log a message with a color.
log = (message, color, explanation) ->
  console.log color + message + reset + ' ' + (explanation or '')

# Walk the filesystem, executing the callback whenever a file is reached.
walk = (filename, callback) ->
  fs.stat filename, (err, stats) ->
    if stats.isFile()
      callback filename
    else if stats.isDirectory()
      fs.readdir filename, (err, files) ->
        walk "#{filename}/#{file}", callback for file in files

# Walk the build files, ignoring minified files.
walkBuildFiles = (callback) ->
  walk 'build', (file) ->
    callback file if file.indexOf(".min.") == -1

task 'build', 'compile and minify CoffeeScript', (options) ->
  console.log 'Compiling CoffeeScript...'
  exec([
    'rm -rf build',
    'cp -r lib build',
    'coffee -c -l -b build',
    'rm build/**/*.coffee'
  ].join(' && '))
  
  invoke 'min'

task 'clean', 'remove all the compiled JavaScript files', (options) ->
  exec 'rm -rf build'

task 'min', 'minify the compiled JavaScript files', (options) ->
  exec 'mkdir -p build', ->
    walkBuildFiles (uncompressedFilename) ->
      minifiedFilename = uncompressedFilename.replace '.js', '.min.js'
      console.log "Minifying #{red}#{uncompressedFilename}#{reset} to #{green}#{minifiedFilename}#{reset}."
      exec "node_modules/.bin/uglifyjs -o #{minifiedFilename} #{uncompressedFilename}"

task 'lint', 'run JSLint on the compiled JavaScript files', (options) ->
  walkBuildFiles (file) ->
    exec "node_modules/.bin/jslint #{file}", (err, stdout, stderr) ->
      console.log "Running JSLint on #{green}#{file}#{reset}:"
      console.log stdout