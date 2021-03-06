###
UNTV Cakefile Tasks
###

request       = require "request"
fs            = require "fs"
os            = require "os"
path          = require "path"
{exec, spawn} = require "child_process"
readline      = require "readline"

################################################################################
### Globals
################################################################################
current_build = "nw-0.8.4-custom"

platforms  = 
  linux32: """
    #{__dirname}/bin/#{current_build}/linux32/nw
  """
  linux64: """
    #{__dirname}/bin/#{current_build}/linux64/nw
  """
  darwin32: """
    "#{__dirname}"/bin/#{current_build}/darwin32/nw.app/Contents/MacOS/node-webkit
  """
  win32: """
    "#{__dirname}"\\bin\\#{current_build}\\win32\\nw
  """

downloads =
  linux32: "" # not built yet
  linux64: "https://file.ac/Ayxb2UM40pg/linux64.tar.gz"
  darwin32: "https://file.ac/8v7SRJ7Xa6A/darwin32.tar.gz"
  win32: "https://file.ac/0FNox66LTbA/win32.zip" 

################################################################################
### Options
################################################################################
option "-p", "--platform [name]", "platform: (linux64|linux32|darwin32|win32)"
option "-o", "--output [dir]", "output directory for build task"
option "-f", "--force", "i sure hope you know what you are doing"

################################################################################
### Setup Custom Node-Webkit Build
################################################################################
task 'setup', 'downloads node-webkit custom build for platform', (options) ->
  platform   = (options.platform or os.platform()).match(/^[A-z]+/)[0]
  only_32    = platform is "win" or platform is "darwin"
  arch       = if only_32 then "32" else os.arch().match /\d+/
  platform   = "#{platform}#{arch}"
  binary_loc = downloads[platform]

  # input checking
  if platform not of platforms
    console.error "'#{platform}' is not supported."
    process.exit -1

  if not binary_loc
    console.log "'#{platform}' custom nw binary not yet available."
    process.exit -1

  if options.force
    console.log "Using --force flag. I sure hope you know what you are doing!\n"

  # download archive
  filename    = path.basename binary_loc
  tmp_loc     = "#{os.tmpdir()}/#{filename}"
  destination = "#{__dirname}/bin/#{current_build}"
  archive     = fs.createWriteStream tmp_loc
  download    = request binary_loc
  bytes_recd  = 0
  # alert user if there is already a build downloaded
  if (fs.existsSync destination) and not options.force
    console.error """
      There is already a Node-Webkit build located at #{destination}.
      If you wish to blast it and download a new version, use the --force flag.
    """
    process.exit -1
  # create readline interface
  rl = readline.createInterface
    input: process.stdin
    output: process.stdout
  # pipe to tmpdir
  console.log """
    Downloading #{current_build} for #{platform} to #{tmp_loc}...
    This might take a minute, hang tight!\n
  """
  rl.write "Bytes Received: #{bytes_recd}"
  # pipe download to file
  download.pipe archive
  # show download indicator
  download.on "data", (chunk) -> 
    rl.write null,  { ctrl: yes, name: "u" }
    bytes_recd = bytes_recd + chunk.length
    rl.write "Bytes Received: #{bytes_recd}"
  # close realine
  download.on "end", -> rl.close()
  # show errors
  download.on "error", (err) -> 
    console.log "Error dowloading #{custom_build}: #{err}"
  # extract archive to bin/#{build}
  archive.on "finish", -> 
    console.log "\nGot it! Extracting archive to #{destination}..."
    # run tar command
    command = """
      tar -xvf #{tmp_loc} -C "#{destination}"
    """
    # check if we are on windows... these fools gotta unzip the files on their
    # own, since we don't have a solid way of knowing how to extract archive
    unless platform is "win32"
      exec command, (err, stdout, stderr) ->
        if err then throw "Error unpacking #{current_build} build: #{err}"
        console.log stdout
        console.log "Setup complete! Run `cake start` to launch UNTV."
    else
      console.log """
        Archive downloaded to #{tmp_loc}! Windows users need to extract this 
        archive to #{destination} before running `cake start` to launch UNTV.
      """

################################################################################
### Start UNTV Using this Platform's Executable
################################################################################
task 'start', 'starts untv application', (options) ->
  platform   = options.platform or os.platform()
  only_32    = platform is "win" or platform is "darwin"
  arch       = if only_32 then "32" else os.arch().match /\d+/
  platform   = "#{platform}#{arch}"
  binary_loc = platforms[platform]

  # input checking
  if platform not of platforms 
    console.log "'#{platform}' is not supported."
    process.exit -1

  if not binary_loc
    console.log "'#{platform}' custom nw binary not yet available."
    process.exit -1

  # spawn the untv process
  command = """
    #{binary_loc} "#{__dirname}"
  """
  untv = exec command
  # pipe untv output to console
  untv.stdout.on "data", (data) -> console.log "untv: #{data}"
  untv.stderr.on "data", (data) -> console.error "untv: #{data}"
  untv.on "close", (code) -> console.log "untv: exited with code #{code}"
  untv.on "error", (err) -> console.log "untv: #{err}"

################################################################################
### Package UNTV Bundle for Specified Platform
################################################################################
task 'build', 'builds platform specific package(s) for untv', (options) ->
  console.log "Build task not yet available."
