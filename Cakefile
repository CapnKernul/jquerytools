fs = require 'fs'
{spawn, exec} = require 'child_process'

directories =
  lib: 'lib'
  test: 'test'
  build: 'build'

commands =
  cleanBuildDir: "rm -rf #{directories.build}"
  createBuildDir: "mkdir -p #{directories.build}"
  build: "coffee -cb -o #{directories.build} #{directories.lib}"
  watch: "coffee -cbw -o #{directories.build} #{directories.lib}"
  test: "expresso --include #{directories.lib} #{directories.test}/*"
  min: (original, minified) -> "uglifyjs -o #{minified} #{original}"

# Use the npm bin directory.
process.env['PATH'] = "node_modules/.bin:#{process.env['PATH']}"

# Walks the filesystem, executing the callback whenever a file is reached.
walk = (filename, callback) ->
  fs.stat filename, (err, stats) ->
    if stats.isFile()
      callback filename
    else if stats.isDirectory()
      fs.readdir filename, (err, files) ->
        walk("#{filename}/#{file}", callback) for file in files

# Walks the build files, ignoring minified files.
walkBuildFiles = (callback) ->
  walk buildDir, (file) ->
    callback(file) if file.indexOf('.min.') == -1

# Executes multiple commands in serial.
serialExec = (commands, err, stdout, stderr) ->
  exec(commands.join(' && '), err, stdout, stderr)

# Executes a command, replacing the current process with the spawned one.
replaceProcess = (command) ->
  segments = command.split(' ')
  childProcess = spawn(segments[0], segments[1..-1])
  process.stdin.on 'data', (data) -> childProcess.stdin.write(data)
  childProcess.stdout.on 'data', (data) -> process.stdout.write(data)
  childProcess.stderr.on 'data', (data) -> process.stderr.write(data)

task 'build', 'compile the CoffeeScript into the build directory', ->
  serialExec([commands.createBuildDir, commands.build])

task 'watch', 'watch the CoffeeScript files for changes, compiling on modification', ->
  replaceProcess(commands.watch)

task 'clean', 'remove all the compiled JavaScript files', ->
  exec(commands.cleanBuildDir)

task 'min', 'minify the compiled JavaScript files', ->
  exec commands.createBuildDir, ->
    walkBuildFiles (original) ->
      minified = orginal.replace('.js', '.min.js')
      exec(commands.min(original, minified))

task 'test', 'run all tests', ->
  replaceProcess(commands.test)