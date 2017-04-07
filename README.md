# api documentation for  [say (v0.11.0)](https://github.com/Marak/say.js#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-say.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-say) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-say.svg)](https://travis-ci.org/npmdoc/node-npmdoc-say)
#### TTS (Text To Speech) Module for Node.js

[![NPM](https://nodei.co/npm/say.png?downloads=true)](https://www.npmjs.com/package/say)

[![apidoc](https://npmdoc.github.io/node-npmdoc-say/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-say_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-say/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-say/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-say/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Marak Squires"
    },
    "bugs": {
        "url": "https://github.com/Marak/say.js/issues"
    },
    "dependencies": {},
    "description": "TTS (Text To Speech) Module for Node.js",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "d4ec524f96a066043956ce62e78a5792a42e47d9",
        "tarball": "https://registry.npmjs.org/say/-/say-0.11.0.tgz"
    },
    "engines": {
        "node": ">=0.10.0"
    },
    "gitHead": "602b30286721f76e4f04255721d45e941b38da08",
    "homepage": "https://github.com/Marak/say.js#readme",
    "license": "MIT",
    "maintainers": [
        {
            "name": "marak",
            "email": "marak.squires@gmail.com"
        },
        {
            "name": "tlhunter",
            "email": "tlhunter@gmail.com"
        }
    ],
    "name": "say",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/Marak/say.js.git"
    },
    "scripts": {},
    "version": "0.11.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module say](#apidoc.module.say)
1.  [function <span class="apidocSignatureSpan">say.</span>export (text, voice, speed, filename, callback)](#apidoc.element.say.export)
1.  [function <span class="apidocSignatureSpan">say.</span>speak (text, voice, speed, callback)](#apidoc.element.say.speak)
1.  [function <span class="apidocSignatureSpan">say.</span>stop (callback)](#apidoc.element.say.stop)
1.  number <span class="apidocSignatureSpan">say.</span>base_speed
1.  string <span class="apidocSignatureSpan">say.</span>speaker



# <a name="apidoc.module.say"></a>[module say](#apidoc.module.say)

#### <a name="apidoc.element.say.export"></a>[function <span class="apidocSignatureSpan">say.</span>export (text, voice, speed, filename, callback)](#apidoc.element.say.export)
- description and source-code
```javascript
export = function (text, voice, speed, filename, callback) {
  var commands, pipedData;

  if (!text) {
    // throw TypeError because API was used incorrectly
    throw new TypeError('Must provide text parameter');
  }

  if (!filename) {
    // throw TypeError because API was used incorrectly
    throw new TypeError('Must provide a filename');
  }

  if (typeof callback !== 'function') {
    callback = function() {};
  }

  // tailor command arguments to specific platforms
  if (process.platform === 'darwin') {

    if (!voice) {
      commands = [ text ];
    } else {
      commands = [ '-v', voice, text];
    }

    if (speed) {
      commands.push('-r', convertSpeed(speed));
    }

    if (filename){
        commands.push('-o', filename, '--data-format=LEF32@32000');
    }
  }  else {
    // if we don't support the platform, callback with an error (next tick) - don't continue
    return process.nextTick(function() {
      callback(new Error('say.js export does not support platform ' + process.platform));
    });
  }

  childD = child_process.spawn(say.speaker, commands);

  childD.stdin.setEncoding('ascii');
  childD.stderr.setEncoding('ascii');

  if (pipedData) {
    childD.stdin.end(pipedData);
  }

  childD.stderr.once('data', function(data) {
    // we can't stop execution from this function
    callback(new Error(data));
  });

  childD.addListener('exit', function (code, signal) {
    if (code === null || signal !== null) {
      return callback(new Error('say.js: could not talk, had an error [code: ' + code + '] [signal: ' + signal + ']'));
    }

    childD = null;

    callback(null);
  });
}
```
- example usage
```shell
...
    return console.error(err);
  }

  console.log('Text has been spoken.');
});

// Export spoken audio to a WAV file
say.export("I'm sorry, Dave.", 'Cellos', 0.75, 'hal.wav', function(err) {
  if (err) {
    return console.error(err);
  }

  console.log('Text has been saved to hal.wav.');
});
'''
...
```

#### <a name="apidoc.element.say.speak"></a>[function <span class="apidocSignatureSpan">say.</span>speak (text, voice, speed, callback)](#apidoc.element.say.speak)
- description and source-code
```javascript
speak = function (text, voice, speed, callback) {
  var commands, pipedData;

  if (typeof callback !== 'function') {
    callback = function() {};
  }

  if (!text) {
    // throw TypeError because API was used incorrectly
    throw new TypeError('Must provide text parameter');
  }

  // tailor command arguments to specific platforms
  if (process.platform === 'darwin') {
    if (!voice) {
      commands = [ text ];
    } else {
      commands = [ '-v', voice, text];
    }

    if (speed) {
      commands.push('-r', convertSpeed(speed));
    }
  } else if (process.platform === 'linux') {
    commands = ['--pipe'];

    if (speed) {
      pipedData = '(Parameter.set \'Audio_Command "aplay -q -c 1 -t raw -f s16 -r $(($SR*' + convertSpeed(speed) + '/100)) $FILE
") ';
    }

    if (voice) {
      pipedData += '(' + voice + ') ';
    }

    pipedData += '(SayText \"' + text + '\")';
  } else if (process.platform === 'win32') {
    pipedData = text;
    commands = [ 'Add-Type -AssemblyName System.speech; $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer; $speak.Speak
([Console]::In.ReadToEnd())' ];
  } else {
    // if we don't support the platform, callback with an error (next tick) - don't continue
    return process.nextTick(function() {
      callback(new Error('say.js speak does not support platform ' + process.platform));
    });
  }

  var options = (process.platform === 'win32') ? { shell: true } : undefined;
  childD = child_process.spawn(say.speaker, commands, options);

  childD.stdin.setEncoding('ascii');
  childD.stderr.setEncoding('ascii');

  if (pipedData) {
    childD.stdin.end(pipedData);
  }

  childD.stderr.once('data', function(data) {
    // we can't stop execution from this function
    callback(new Error(data));
  });

  childD.addListener('exit', function (code, signal) {
    if (code === null || signal !== null) {
      return callback(new Error('say.js: could not talk, had an error [code: ' + code + '] [signal: ' + signal + ']'));
    }

    childD = null;

    callback(null);
  });
}
```
- example usage
```shell
...

## Usage

'''javascript
var say = require('say');

// Use default system voice and speed
say.speak('Hello!');

// Stop the text currently being spoken
say.stop();

// More complex example (with an OS X voice) and slow speed
say.speak('whats up, dog?', 'Alex', 0.5);
...
```

#### <a name="apidoc.element.say.stop"></a>[function <span class="apidocSignatureSpan">say.</span>stop (callback)](#apidoc.element.say.stop)
- description and source-code
```javascript
stop = function (callback) {
  if (typeof callback !== 'function') {
    callback = function() {};
  }

  if (!childD) {
    return callback(new Error('No speech to kill'));
  }

  if (process.platform === 'linux') {
    // TODO: Need to ensure the following is true for all users, not just me. Danger Zone!
    // On my machine, original childD.pid process is completely gone. Instead there is now a
    // childD.pid + 1 sh process. Kill it and nothing happens. There's also a childD.pid + 2
    // aplay process. Kill that and the audio actually stops.
    process.kill(childD.pid + 2);
  } else if (process.platform === 'win32') {
    childD.stdin.pause();
    child_process.exec('taskkill /pid ' + childD.pid + ' /T /F')
  } else {
    childD.stdin.pause();
    childD.kill();
  }

  childD = null;

  callback(null);
}
```
- example usage
```shell
...
'''javascript
var say = require('say');

// Use default system voice and speed
say.speak('Hello!');

// Stop the text currently being spoken
say.stop();

// More complex example (with an OS X voice) and slow speed
say.speak('whats up, dog?', 'Alex', 0.5);

// Fire a callback once the text has completed being spoken
say.speak('whats up, dog?', 'Good News', 1.0, function(err) {
if (err) {
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
