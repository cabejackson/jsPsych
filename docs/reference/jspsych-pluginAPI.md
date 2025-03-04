# jsPsych.pluginAPI

The pluginAPI module contains functions that are useful when developing new plugins.

---

## jsPsych.pluginAPI.cancelAllKeyboardResponses

```javascript
jsPsych.pluginAPI.cancelAllKeyboardResponses()
```

### Parameters

None.

### Return value

Returns nothing.

### Description

Cancels all currently active keyboard listeners created by `jsPsych.pluginAPI.getKeyboardResponse`.

### Example

```javascript
jsPsych.pluginAPI.cancelAllKeyboardResponses();
```

---

## jsPsych.pluginAPI.cancelKeyboardResponse

```javascript
jsPsych.pluginAPI.cancelKeyboardResponse(listener_id)
```

### Parameters

Parameter | Type | Description
----------|------|------------
listener_id | object | The listener_id object generated by the call to `jsPsych.pluginAPI.getKeyboardResponse`.

### Return value

Returns nothing.

### Description

Cancels a specific keyboard listener created by `jsPsych.pluginAPI.getKeyboardResponse`.

### Example

```javascript
// create a persistent keyboard listener
var listener_id = jsPsych.pluginAPI.getKeyboardResponse({
    callback_function: after_response, 
    valid_responses: ['p','q'], 
    rt_method: 'performance', 
    persist: true,
    allow_held_key: false
});

// cancel keyboard listener
jsPsych.pluginAPI.cancelKeyboardResponse(listener_id);
```

---

## jsPsych.pluginAPI.clearAllTimeouts

```javascript
jsPsych.pluginAPI.clearAllTimeouts()
```

### Parameters

None.

### Return value

Returns nothing.

### Description

Clears any pending timeouts that were set using jsPsych.pluginAPI.setTimeout().

---

## jsPsych.pluginAPI.compareKeys

```javascript
jsPsych.pluginAPI.compareKeys(key1, key2)
```

### Parameters

Parameter | Type | Description
----------|------|------------
key1 | string or numeric | The representation of a key, either string or keycode
key2 | string or numeric | The representation of a key, either string or keycode

### Return value

Returns true if keycodes or strings refer to the same key, regardless of type. Returns false if the keycodes or strings do not match.

### Description

Compares two keys to see if they are the same, ignoring differences in representational type, and using the appropriate case sensitivity based on the experiment's settings. 

If `case_sensitive_responses` is set to `false` in `initJsPsych` (the default), then the string key comparison will not be case-sensitive, e.g., "a" and "A" will match, and this function will return `true`. If `case_sensitive_responses` is set to `true` in `initJsPsych`, then the string key comparison will not be case-sensitive, e.g., "a" and "A" will not match, and this function will return `false`. 

We recommend using this function to compare keys in all plugin and experiment code, rather than using something like `if (response == 'j')...`. This is because the response key returned by the `jsPsych.pluginAPI.getKeyboardResponse` function will be converted to lowercase when `case_sensitive_responses` is `false`, and it will match the exact key press representation when `case_sensitive_responses` is `true`. Using this `compareKeys` function will ensure that your key comparisons work appropriately based on the experiment's `case_sensitive_responses` setting, and that you do not need to remember to check key responses against different case versions of the comparison key (e.g. `if (response == 'ArrowLeft' || response == 'arrowleft')...`). 

### Examples

#### Basic examples

```javascript
jsPsych.pluginAPI.compareKeys('a', 'A');
// returns true when case_sensitive_responses is false in initJsPsych

jsPsych.pluginAPI.compareKeys('a', 'A');
// returns false when case_sensitive_responses is true in initJsPsych

// also works with numeric key codes (but note that numeric keyCode values are now deprecated)
jsPsych.pluginAPI.compareKeys('a', 65);
// returns true

jsPsych.pluginAPI.compareKeys('space', 31);
// returns false
```

#### Comparing a key response and key parameter value in plugins

```javascript
// this is the callback_function passed to jsPsych.pluginAPI.getKeyboardResponse
var after_response = function(info) {
  // score the response by comparing the key that was pressed against the trial's key_answer parameter
  var correct = jsPsych.pluginAPI.compareKeys(trial.key_answer, info.key);
  //...
}
```

#### Scoring a key response in experiment code

```javascript
var trial = {
  type: jsPsychHtmlKeyboardResponse,
  stimulus: '<<<<<',
  choices: ['f','j'],
  prompt: 'Press f for left. Press j for right.',
  on_finish: function(data){
    // score the response by comparing the key that was pressed (data.response) against the 
    // correct response for this trial ('f'), and store reponse accuracy in the trial data
    if(jsPsych.pluginAPI.compareKeys(data.response, 'f')){
      data.correct = true;
    } else {
      data.correct = false; 
    }
  }
}
```

---

## jsPsych.pluginAPI.getAudioBuffer

```javascript
jsPsych.pluginAPI.getAudioBuffer(filepath)
```

### Parameters

Parameter | Type | Description
----------|------|------------
filepath | string | The path to the audio file that was preloaded.

### Return value

Returns a Promise that resolves when the audio file loads. Success handler's parameter will be the audio buffer. If the experiment is running using the WebAudio API it will be an AudioBuffer object. Otherwise, it will be an HTML5 Audio object. The failure handler's parameter is the error generated by `preloadAudio`.

### Description

Gets an AudioBuffer that can be played with the WebAudio API or an Audio object that can be played with HTML5 Audio. 

It is strongly recommended that you preload audio files before calling this method. This method will load the files if they are not preloaded, but this may result in delays during the experiment as audio is downloaded.

### Examples

#### HTML 5 Audio

```javascript
jsPsych.pluginAPI.getAudioBuffer('my-sound.mp3')
  .then(function(audio){
    audio.play();
  })
  .catch(function(err){
    console.error('Audio file failed to load')
  })
```

#### WebAudio API

```javascript
var context = jsPsych.pluginAPI.audioContext();

jsPsych.pluginAPI.getAudioBuffer('my-sound.mp3')
  .then(function(buffer){
    audio = context.createBufferSource();
    audio.buffer = buffer;
    audio.connect(context.destination);
    audio.start(context.currentTime);
  })
  .catch(function(err){
    console.error('Audio file failed to load')
  })
```

See the `audio-keyboard-response` plugin for an example in a fuller context.

---

## jsPsych.pluginAPI.getAutoPreloadList

```javascript
jsPsych.pluginAPI.getAutoPreloadList(timeline)
```

### Parameters

Parameter | Type | Description
----------|------|------------
timeline | array | An array containing the trial object(s) from which a list of media files should be automatically generated. This array can contain the entire experiment timeline, or any individual parts of a larger timeline, such as specific timeline nodes and trial objects.

### Return value

An object with properties for each media type: `images`, `audio`, and `video`. Each property contains an array of the unique files of that media type that were automatically extracted from the timeline. If no files are found in the timeline for a particular media type, then the array will be empty for that type.

### Description

This method is used to automatically generate lists of unique image, audio, and video files from a timeline. It is used by the `preload` plugin to generate a list of to-be-preloaded files based on the trials passed to the `trials` parameter and/or the experiment timeline passed to `jsPsych.run` (when `auto_preload` is true). It can be used in custom plugins and experiment code to generate a list of audio/image/video files, based on a timeline. 

This function will only return files from plugin parameters that are marked as parameter type `AUDIO`/`IMAGE`/`VIDEO`, and only when the `preload` flag of the corresponding parameter definition has not been set to `false`, and the trial's parameter value is not a function.
When a file path is returned to the trial parameter from a function (including the `jsPsych.timelineVariable` function), or when the file path is embedded in an HTML string, that file will not be detected by the `getAutoPreloadList` method.
In these cases, the file should be preloaded manually.
See [Media Preloading](../overview/media-preloading.md) for more information.

### Example

```javascript
var audio_trial = {
    type: jsPsychAudioKeyboardResponse
    stimulus: 'file.mp3'
}

var image_trial = {
    type: jsPsychImageKeyboardResponse
    stimulus: 'file.png'
}

var video_trial = {
    type: jsPsychVideoKeyboardResponse
    stimulus: 'file.mp4'
}

var timeline = [audio_trial, image_trial, video_trial];

jsPsych.pluginAPI.getAutoPreloadList(timeline);
```

---

## jsPsych.pluginAPI.getKeyboardResponse

```javascript
jsPsych.pluginAPI.getKeyboardResponse(parameters)
```

### Parameters

The method accepts an object of parameter values (see example below). The valid keys for this object are listed in the table below.

Parameter | Type | Description
----------|------|------------
callback_function | function | The function to execute whenever a valid keyboard response is generated.
valid_responses | array | An array of key codes or character strings representing valid responses. Responses not on the list will be ignored. An empty array indicates that all responses are acceptable.
rt_method | string | Indicates which method of recording time to use. The `'performance'` method uses calls to `performance.now()`, which is the standard way of measuring timing in jsPsych. It is [supported by up-to-date versions of all the major browsers](http://caniuse.com/#search=performance). The `audio` method is used in conjuction with an `audio_context` (set as an additional parameter). This uses the clock time of the `audio_context` when audio stimuli are being played.
audio_context | AudioContext object | The AudioContext of the audio file that is being played.
audio_context_start_time | numeric | The scheduled time of the sound file in the AudioContext. This will be used as the start time.
allow_held_key | boolean | If `true`, then responses will be registered from keys that are being held down. If `false`, then a held key can only register a response the first time that `getKeyboardResponse` is called for that key. For example, if a participant holds down the `A` key before the experiment starts, then the first time `getKeyboardResponse` is called, the `A` will register as a key press. However, any future calls to `getKeyboardResponse` will not register the `A` until the participant releases the key and presses it again.
persist | boolean | If false, then the keyboard listener will only trigger the first time a valid key is pressed. If true, then it will trigger every time a valid key is pressed until it is explicitly cancelled by `jsPsych.pluginAPI.cancelKeyboardResponse` or `jsPsych.pluginAPI.cancelAllKeyboardResponses`.

### Return value

Return an object that uniquely identifies the keyboard listener. This object can be passed to `jsPsych.pluginAPI.cancelKeyboardResponse` to cancel the keyboard listener.

### Description

Gets a keyboard response from the subject, recording the response time from when the function is first called until a valid response is generated.

The keyboard event listener will be bound to the `display_element` declared in `initJsPsych()` (or the `<body>` element if no `display_element` is specified). This allows jsPsych experiments to be embedded in websites with other content without disrupting the functionality of other UI elements.

A valid response triggers the `callback_function` specified in the parameters. A single argument is passed to the callback function. The argument contains an object with the properties `key` and `rt`. `key` contains the string representation of the response key, and `rt` contains the response time. 

This function uses the `.key` value of the keyboard event, which is _case sensitive_. When `case_sensitive_responses` is `false` in `initJsPsych` (the default), this function will convert both the `valid_responses` strings and the response key to lowercase before comparing them, and it will pass the lowercase version of the response key to the `callback_function`. For example, if `valid_responses` is `['a']`, then both 'A' and 'a' will be considered valid key presses, and 'a' will be returned as the response key. When `case_sensitive_responses` is `true` in `initJsPsych`, this function will not convert the case when comparing the `valid_responses` and response key, and it will not convert the case of the response key that is passed to the `callback_function`. For example, if `valid_responses` is `['a']`, then 'a' will be the only valid key press, and 'A' (i.e. 'a' with CapsLock on or Shift held down) will not be accepted. Also, if `valid_responses` includes multiple letter case options (e.g. `"ALL_KEYS"`), then you may need to check the response key against both letter cases when scoring etc., e.g. `if (response == 'ArrowLeft' || response =='arrowleft') ...`.

### Examples

#### Get a single response from any key

```javascript

var after_response = function(info){
	alert('You pressed key '+info.key+' after '+info.rt+'ms');
}

jsPsych.pluginAPI.getKeyboardResponse({
  callback_function:after_response,
  valid_responses: "ALL_KEYS",
  rt_method: 'performance',
  persist: false
});
```

#### Get a responses from a key until the letter q is pressed

```javascript

var after_response = function(info){
	alert('You pressed key '+info.key+' after '+info.rt+'ms');

	if(jsPsych.pluginAPI.compareKeys(info.key,'q')){ /
		jsPsych.pluginAPI.cancelKeyboardResponse(listener);
	}
}

var listener = jsPsych.pluginAPI.getKeyboardResponse({
  callback_function:after_response,
  valid_responses: "ALL_KEYS",
  rt_method: 'performance',
  persist: true
});
```

---

## jsPsych.pluginAPI.preloadAudio

```javascript
jsPsych.pluginAPI.preloadAudio(files, callback_complete, callback_load, callback_error)
```

### Parameters

Parameter | Type | Description
----------|------|------------
files | array | An array of audio file paths to load. The array can be nested (e.g., if images are in multiple arrays to help sort by condition or task).
callback_complete | function | A function to execute when all the files have been loaded.
callback_load | function | A function to execute after a single file has been loaded. A single parameter is passed to this function which is the file source (string) that has loaded.
callback_error | function | A function to execute after a single file has produced an error. A single parameter is passed to this function which is the file source (string) that produced the error.

### Return value

Returns nothing.

### Description

This function is used to preload audio files. It is used by the `preload` plugin, and could be called directly to preload audio files in custom plugins or experiment. See [Media Preloading](../overview/media-preloading.md) for more information.

It is possible to run this function without specifying a callback function. However, in this case the code will continue executing while the files are loaded. Thus, it is possible that an audio file would be required for playing before it is done preloading. The `callback_complete` function will only execute after all the audio files are loaded, and can be used to control the flow of the experiment (e.g., by starting the experiment in the `callback_complete` function).

The `callback_load` and `callback_error` functions are called after each file has either loaded or produced an error, so these functions can also be used to monitor loading progress. See example below.

### Examples

#### Basic use

```javascript
var sounds = ['file1.mp3', 'file2.mp3', 'file3.mp3'];

jsPsych.pluginAPI.preloadAudio(sounds, 
    function(){ startExperiment(); },
    function(file){ console.log('file loaded: ', file); }
    function(file){ console.log('error loading file: ', file); }
);

function startExperiment(){
    jsPsych.run(exp);
}
```

#### Show progress of loading

```javascript
var sounds = ['file1.mp3', 'file2.mp3', 'file3.mp3'];
var n_loaded = 0;

jsPsych.pluginAPI.preloadAudio(sounds, function(){ startExperiment(); }, function(file) { updateLoadedCount(file); });

function updateLoadedCount(file){
  n_loaded++;
	var percentcomplete = n_loaded / sounds.length * 100;

	// could put something fancier here, like a progress bar
	// or updating text in the DOM.
	console.log('Loaded '+percentcomplete+'% of audio files');
}

function startExperiment(){
  jsPsych.run(exp);
}
```

---

## jsPsych.pluginAPI.preloadImages

```javascript
jsPsych.pluginAPI.preloadImages(images, callback_complete, callback_load, callback_error)
```

### Parameters

Parameter | Type | Description
----------|------|------------
images | array | An array of image paths to load. The array can be nested (e.g., if images are in multiple arrays to help sort by condition or task).
callback_complete | function | A function to execute when all the images have been loaded.
callback_load | function | A function to execute after a single file has been loaded. A single parameter is passed to this function which is the file source (string) that has loaded.
callback_error | function | A function to execute after a single file has produced an error. A single parameter is passed to this function which is the file source (string) that produced the error.

### Return value

Returns nothing.

### Description

This function is used to preload image files. It is used by the `preload` plugin, and could be called directly to preload image files in custom plugins or experiment code. See [Media Preloading](../overview/media-preloading.md) for more information.

It is possible to run this function without specifying a callback function. However, in this case the code will continue executing while the images are loaded. Thus, it is possible that an image would be required for display before it is done preloading. The `callback_complete` function will only execute after all the images are loaded, and can be used to control the flow of the experiment (e.g., by starting the experiment in the `callback_complete` function).

The `callback_load` and `callback_error` functions are called after each file has either loaded or produced an error, so these functions can also be used to monitor loading progress. See example below.

### Examples

#### Basic use

```javascript
var images = ['img/file1.png', 'img/file2.png', 'img/file3.png'];

jsPsych.pluginAPI.preloadImages(images, 
    function(){ startExperiment(); },
    function(file){ console.log('file loaded: ', file); }
    function(file){ console.log('error loading file: ', file); }
);

function startExperiment(){
    jsPsych.run(exp);
}
```

#### Show progress of loading

```javascript
var images = ['img/file1.png', 'img/file2.png', 'img/file3.png'];
var n_loaded = 0;

jsPsych.pluginAPI.preloadImages(images, function(){ startExperiment(); }, function(file) { updateLoadedCount(file); });

function updateLoadedCount(file){
  n_loaded++;
	var percentcomplete = n_loaded / images.length * 100;

	// could put something fancier here, like a progress bar
	// or updating text in the DOM.
	console.log('Loaded '+percentcomplete+'% of images');
}

function startExperiment(){
  jsPsych.run(exp);
}
```

---

## jsPsych.pluginAPI.preloadVideo

```javascript
jsPsych.pluginAPI.preloadVideo(video, callback_complete, callback_load, callback_error)
```

### Parameters

Parameter | Type | Description
----------|------|------------
video | array | An array of video paths to load. The array can be nested (e.g., if videos are in multiple arrays to help sort by condition or task).
callback_complete | function | A function to execute when all the videos have been loaded.
callback_load | function | A function to execute after a single file has been loaded. A single parameter is passed to this function which is the file source (string) that has loaded.
callback_error | function | A function to execute after a single file has produced an error. A single parameter is passed to this function which is the file source (string) that produced the error.

### Return value

Returns nothing.

### Description

This function is used to preload video files. It is used by the `preload` plugin, and could be called directly to preload video files in custom plugins or experiment code. See [Media Preloading](../overview/media-preloading.md) for more information.

It is possible to run this function without specifying a callback function. However, in this case the code will continue executing while the videos are loaded. Thus, it is possible that a video would be requested before it is done preloading. The `callback_complete` function will only execute after all the videos are loaded, and can be used to control the flow of the experiment (e.g., by starting the experiment in the `callback_complete` function).

The `callback_load` and `callback_error` functions are called after each file has either loaded or produced an error, so these functions can also be used to monitor loading progress. See example below.

### Examples

#### Basic use

```javascript
var videos = ['vid/file1.mp4', 'vid/file2.mp4', 'vid/file3.mp4'];

jsPsych.pluginAPI.preloadVideo(videos, 
  function(){ startExperiment(); },
  function(file){ console.log('file loaded: ', file); }
  function(file){ console.log('error loading file: ', file); }
);

function startExperiment(){
  jsPsych.run(exp);
}
```

#### Show progress of loading

```javascript
var videos = ['vid/file1.mp4', 'vid/file2.mp4', 'vid/file3.mp4'];
var n_loaded = 0;

jsPsych.pluginAPI.preloadVideo(videos, function(){ startExperiment(); }, function(file) { updateLoadedCount(file); });

function updateLoadedCount(file){
  n_loaded++;
	var percentcomplete = n_loaded / videos.length * 100;

	// could put something fancier here, like a progress bar
	// or updating text in the DOM.
	console.log('Loaded '+percentcomplete+'% of videos');
}

function startExperiment(){
  jsPsych.run(exp);
}
```

---

## jsPsych.pluginAPI.setTimeout

```javascript
jsPsych.pluginAPI.setTimeout(callback, delay)
```

### Parameters

Parameter | Type | Description
----------|------|------------
callback | function | A function to execute after waiting for delay.
delay | integer | Time to wait in milliseconds.

### Return value

Returns the ID of the setTimeout handle.

### Description

This is simply a call to the standard setTimeout function in JavaScript with the added benefit of registering the setTimeout call in a central list. This is useful for scenarios where some other event (the trial ending, aborting the experiment) should stop the execution of queued timeouts.

### Example

```javascript
// print the time
console.log(Date.now())

// print the time 1s later
jsPsych.pluginAPI.setTimeout(function(){
	console.log(Date.now())
}, 1000);
```
