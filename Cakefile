fs = require 'fs'
path = require 'path'

CoffeeScript = require 'coffee-script'
express = require 'express'
{parser, uglify} = require 'uglify-js'


files = [
  'ga.js'
  'prettify.js'
  'base64/base64.js'
  'showdown/lib/showdown.js'
  'showdown/lib/datetimes.js'
  'showdown/lib/abbreviations.js'
  'hashify-editor/hashify-editor.js'
  'src/hashify.coffee'
]

option '-o', '--output [FILE]', 'write output to <file> instead of stdout'

task 'build:scripts', 'concatenate and minify JavaScript files', (options) ->
  read = (filename) ->
    data = fs.readFileSync filename, 'utf8'
    if /[.]coffee$/.test filename then CoffeeScript.compile data else data

  {ast_mangle, ast_squeeze, gen_code} = uglify
  data = (read f for f in files).join(';')
  data = gen_code ast_squeeze ast_mangle parser.parse data

  if options.output? then fs.writeFileSync options.output, data, 'utf8'
  else console.log data

task 'server', 'start the development server', ->
  serve = (fn) -> (req, res) ->
    filename = fn req
    [first, second, rest..., last] = filename.split '/'
    if compile = first is '' and second is 'lib'
      last = last.replace /[.]js$/, '.coffee'
      filename = [first, 'src', rest..., last].join '/'

    filename = __dirname + filename
    path.exists filename, (exists) ->
      if exists
        if compile
          res.contentType 'js'
          res.send CoffeeScript.compile fs.readFileSync filename, 'utf8'
        else
          res.sendfile filename
      else
        res.redirect '/' + Buffer('# 400 Bad Request').toString('base64')

  app = express.createServer()
  app.get /^\/[A-Za-z0-9+/=]*$/, serve -> '/index.html'
  app.get /^\/unpack:[A-Za-z0-9]+(?:,[A-Za-z0-9]+)*$/, serve -> '/index.html'
  app.get '*', serve (req) -> req.route.params[0]

  port = process.env.PORT ? 3000
  app.listen port, -> console.log "listening on port #{port}"
