JSBIN
https://jsbin.com/deritexoya/edit?html,css,js,console,output

HTML
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <div class="eye"></div>
  <!-- Cozmo-like expressions testing -->
  <canvas id="myCanvas"></canvas>
</body>
</html>
```

CSS
```css
.eye {
  display: block;
  position: absolute;
  height: 10px;
  width: 10px;
  left: 0px;
  top: 0px;
  background: red;
  display: none;
}

#myCanvas {
  border: 1px solid black;
  display: block;
  position: absolute;
}
```

JS
```javascript
// quick sprite-based animation on Espruino
var anims = {
  'action1': [
    new Uint8Array([
      0b00000000,
      0b01000010,
      0b10100101,
      0b00000000,
      0b10011001,
      0b01000010,
      0b00111100,
      0b00000000,
    ]),//.buffer,
    new Uint8Array([
      1, //0b11111111,
      2, //0b11111111,
      3, //0b11000011,
      4, //0b11000011,
      5, //0b11000011,
      6, //0b11000011,
      7, //0b11111111,
      8 //0b11111111,
    ]),//.buffer,
    new Uint8Array([
      9, //0b11111111,
      10, //0b11000011,
      11, //0b11000011,
      12, //0b11000011,
      13, //0b11000011,
      14, //0b11000011,
      15, //0b11000011,
      16 //0b11111111,
    ]), //.buffer,
    new Uint8Array([
      17, //0b11111111,
      18, //0b11000011,
      19, //0b11000011,
      20, //0b11000011,
      21, //0b11000011,
      22, //0b11000011,
      23, //0b11000011,
      24 //0b11111111,
    ]), //.buffer,
    new Uint8Array([
      0b00000000,
      0b01000010,
      0b10100101,
      0b00000000,
      0b10011001,
      0b01000010,
      0b00111100,
      0b00000000,
    ]),//.buffer,
  ]
}

// type of animation:
// - what to do on anim end
// - looping indefinately, n times or not at all
// - if looping, from end to beginng or forth & back
var animEnd = function(){ console.log('animation complete'); }
var looping = 0; // -1 == infinite, 0 == not looping, n == loop times
var animLoopType = 0; // 0 == beginning to end & over, 1 == forth & back
var animName = 'action1'; // the anim name ( if we have more than one anim )
var animDir = 0; // 0 == forward, 1 == backward 
var animIdx = 0; // the step we're in for a particular anim
var animRate = 1000; // the speed of our anim
var animated = true; // whether or not to continue animating at animRate intervals
var animItr = -1; // the interval used for refreshing & stepping through the anim
var animate = function(callback){
  console.log('animate() called ..');
  if(animated === true){
    console.log('animated is true ..');
    // check end of anim depending on animLoopType ( end of anim & not looping ? callback if any & set "animate" to false )
    if(animLoopType == 1 && animDir == 1 && looping === 0 && animIdx === 0){
      // back & forth non-infinite anim just ended
      //refreshDisplay4(); // last refresh
      refreshDisplay5();
      console.log('back & forth non-infinite anim just ended: animIdx ==' + animIdx);
      animated = false;
      if(callback) callback();
      clearInterval(animItr);
      animItr = -1;
    } else if (animLoopType === 0 && looping === 0 && anims[animName].length-1 == animIdx){
      // beginning to end & over non-infinite anim just ended
      //refreshDisplay4(); // last refresh
      refreshDisplay5();
      console.log('beginning to end & over non-infinite anim just ended: animIdx ==' + animIdx);
      animated = false;
      if(callback) callback();
      clearInterval(animItr);
      animItr = -1;
    // TODO: end of anim & looping ? if not infinite, decrease loop times & act depending on loop type
    } else { // not end of anim ? continue stepping through the anim base on animDir & animIdx
      console.log('not end of animation: animIdx ==' + animIdx);
      // else, update based on stuff
      if (animLoopType === 0 && animIdx <= anims[animName].length ){ animIdx++; }
      else if (animLoopType === 1 && animDir === 0 && anims[animName].length-1 == animIdx ){ animIdx--; animDir = 1; }
      else if (animLoopType === 1 && animDir === 0 && animIdx < anims[animName].length ){ animIdx++; }
      else if (animLoopType === 1 && animDir === 1 && animIdx > 0 ){ animIdx--; }
    
      // refresh the display
      //refreshDisplay(); // debug helper
      //console.clear();
      //refreshDisplay4();
      refreshDisplay5();
      //animated = false; // debug
      // way A: new Uint8Array(g.buffer).set(anims[animName][animIdx]); // for full screen anims
      // way B: g.drawImage({width: 8, height: 8, bpp: 1, buffer: anims[animName][animIdx], x, y});
      // g.flip();
    }
  } else {
    console.log('animated is false ..');
    // stop anim & reset anim itr
    //clearInterval(animItr);
    //animItr = -1;
  }
  
  // fail switch ;p
  //animated = false;
  //clearInterval(animItr);
}

// ---- DISPLAY HELPERS, aka FAKE SCREEN IN CONSOLE ;P ----
// our display helper
var refreshDisplay = function(){
  var logStr = '( %c';
    for(var i=0; i< animIdx; i++){ logStr+= '['+ anims[animName][i] + '] '; } // add items before
    logStr += ') %c['; logStr += anims[animName][animIdx]; logStr += '] %c(';
    for(var i=animIdx+1; i< anims[animName].length; i++){ logStr+= '['+ anims[animName][i] + '] '; } // add items after
    logStr += ');'
    console.log(logStr, 'color: black;', 'color: blue;', 'color: black;');
}
// modded version of the above to display arrays as block rows
var refreshDisplay2 = function(){
  var logStr = '%c';
    for(var i=0; i< animIdx; i++){ logStr+= ''+ anims[animName][i] + '\n'; } // add items before
    logStr += '%c'; logStr += anims[animName][animIdx]; logStr += '%c\n';
    for(var i=animIdx+1; i< anims[animName].length; i++){ logStr+= ''+ anims[animName][i] + '\n'; } // add items after
    logStr += ''
    console.log(logStr, 'color: black;', 'color: blue;', 'color: black;');
}
// modded version of the above to display stuff as bins ( TODO: pad then parse )
var refreshDisplay3 = function(){
  var logStr = '%c';
    for(var i=0; i< animIdx; i++){ logStr+= ''+ anims[animName][i].toString(2) + '\n'; } // add items before
    logStr += '%c';
    for(var y=0; y< anims[animName][animIdx].length; y++){ logStr += anims[animName][animIdx][y].toString(2) + '\n'; }
    logStr += '%c\n';
    for(var i=animIdx+1; i< anims[animName].length; i++){ logStr+= ''+ anims[animName][i].toString(2) + '\n'; } // add items after
    logStr += ''
    console.log(logStr, 'color: black;', 'color: blue;', 'color: black;');
}
// "final" helper drawing "pixels" within the console ;P
var styles0 = [
    //'background: gray' // debug
    'background: rgba(255, 255, 255, .7)'
    , 'color: black' // nice for debug/testing & displaying overlaying value
    //, 'color: rgba(255, 255, 255, .7)' // nice for overlaying value - when using gray background
    //, 'color: rgba(0, 0, 0, .7)' // nice for overlaying value - when using white-ish background
    //, 'color: transparent' // when not using overlays
    , 'display: block'
    , 'line-height: 13px'
    , 'text-align: left'
    , 'text-indent: 6.5px'
    , 'font-family: Monospace'
    , 'font-weight: 13px'
].join(';');
var styles1 = [
    'background: black'
    , 'color: white' // nice for debug/testing & displaying overlaying value
    //, 'color: rgba(255, 255, 255, .7)' // nice for overlaying value
    //, 'color: transparent' // when not using overlays
    , 'display: block'
    //, 'text-shadow: 0 1px 0 rgba(0, 0, 0, 0.3)'
    //, 'box-shadow: 0 1px 0 rgba(255, 255, 255, 0.4) inset, 0 5px 3px -5px rgba(0, 0, 0, 0.5), 0 -13px 5px -10px rgba(255, 255, 255, 0.4) inset'
    , 'line-height: 13px'
    , 'text-align: left'
    , 'text-indent: 6.5px'
    //, 'font-weight: bold'
    , 'font-family: Monospace'
    , 'font-weight: 13px'
].join(';');

//console.log('%c0b1111%c1111\n0b1111%c1111', styles0, styles1, styles0);
//console.log('%c    %c    \n    %c    ', styles0, styles1, styles0);
//console.log('%c  %c  \n  %c  ', styles0, styles1, styles0); // correct "pixel" ratio
//console.log('%c00%c11\n11%c00', styles0, styles1, styles0); // correct "pixel" ratio
//console.log('%c0 %c1 \n1 %c0 ', styles0, styles1, styles0); // correct "pixel" ratio
//console.log('%c0%c1\n1%c0', styles0, styles1, styles0); // correct "pixel" ratio with only one "bin digit" ( 0 or 1 ) <--- THE NEAT ONE
//console.log('%c %c \n %c ', styles0, styles1, styles0);
var paddingBits = 8; // 1 byte padding for now - could == Uint8Array.byteLength ?
//var paddingBits = 16;
var styles = []; //['color: black;'];
var refreshDisplay4 = function(){
  styles = ['color: black;'];
  var logStr = '%c';
    for(var i=0; i< animIdx; i++){ logStr+= ''+ anims[animName][i].toString(2) + '\n'; } // add items before
    //logStr += '%c';
    for(var y=0; y< anims[animName][animIdx].length; y++){
      //logStr += anims[animName][animIdx][y].toString(2) + '\n';
      
      // get, pad & parse
      var binStr = anims[animName][animIdx][y].toString(2);
      if( binStr.length < paddingBits){
        var padding = '';
        for(var p=0; p < paddingBits - binStr.length; p++){ padding += '0'; }
        binStr = padding+binStr; // prefix it with padding
      }
      //logStr += binStr + ' ( unpadded: ' + anims[animName][animIdx][y].toString(2) +')\n'; // works fine
      // loop over padded bin str
      var tmpStr = '';
      for(var s=0; s < binStr.length; s++){
        // if s === 0, add style0, else add style 2
        tmpStr += ('%c' + binStr[s]); // add to logStr
        if(binStr[s] === "0") styles.push(styles0);
        else styles.push(styles1);
      }
      //logStr += tmpStr + ' ( padded' + binStr + 'unpadded: ' + anims[animName][animIdx][y].toString(2) +')\n'; // works fine
      logStr += (tmpStr + '\n');
      
    }
    logStr += '%c\n';
    styles.push('color: black;');
    for(var i=animIdx+1; i< anims[animName].length; i++){ logStr+= ''+ anims[animName][i].toString(2) + '\n'; } // add items after
    logStr += ''
    //console.log(logStr, 'color: black;', 'color: blue;', 'color: black;');
    styles.unshift(logStr); // push logStr at beginning our our style array
    console.log.apply(this, styles); // pass the array "as is" to the console & have it "expanded" as actual args
    return undefined; // prevent filling screen with passed args
    styles = ['color: black;'];
}
// mod of the above displaying only the current anim idx
var refreshDisplay5 = function(){
  styles = [];
  var logStr = '';
    for(var y=0; y< anims[animName][animIdx].length; y++){
      //logStr += anims[animName][animIdx][y].toString(2) + '\n';
      
      // get, pad & parse
      var binStr = anims[animName][animIdx][y].toString(2);
      if( binStr.length < paddingBits){
        var padding = '';
        for(var p=0; p < paddingBits - binStr.length; p++){ padding += '0'; }
        binStr = padding+binStr; // prefix it with padding
      }
      //logStr += binStr + ' ( unpadded: ' + anims[animName][animIdx][y].toString(2) +')\n'; // works fine
      // loop over padded bin str
      var tmpStr = '';
      for(var s=0; s < binStr.length; s++){
        // if s === 0, add style0, else add style 2
        tmpStr += ('%c' + binStr[s]); // add to logStr
        if(binStr[s] === "0") styles.push(styles0);
        else styles.push(styles1);
      }
      //logStr += tmpStr + ' ( padded' + binStr + 'unpadded: ' + anims[animName][animIdx][y].toString(2) +')\n'; // works fine
      logStr += (tmpStr + '\n');
      
    }
    logStr += '%c\n';
    styles.push('color: black;');
    logStr += '\n\n'
    //console.log(logStr, 'color: black;', 'color: blue;', 'color: black;');
    styles.unshift(logStr); // push logStr at beginning our our style array
    console.log.apply(this, styles); // pass the array "as is" to the console & have it "expanded" as actual args
    return undefined; // prevent filling screen with passed args
    styles = [];
}

// use our anim helper
//console.clear();
//refreshDisplay4();

//animIdx = 1;
//refreshDisplay4();
//animIdx = 2;
//refreshDisplay4();
//animIdx = 3;
//refreshDisplay4();

//animItr = setInterval(animate, animRate);
/*
animItr = setInterval(function(){
  animate(animEnd);
}, animRate);
*/





// ---- TWEENING TESTS ----
// http://www.acuriousanimal.com/2010/12/18/javascript-animations-why-maths-are-important.html
var eye = { x1: 0, y1: 0, x2: 0, y2: 0, x3: 0, y3: 0, x4: 0, y4: 0, relOffX: 0, relOffY: 0 } // from which we'll draw a polygon
var nullObj = { x: 0, y: 0}; // from which 1 or 2 eyes 'll get their relative position from

// we start with our nullObj centered
var screenW = 128;
var screenH = 64;
nullObj.x = screenW/2;
nullObj.y = screenH/2;
// we set our eye X & Y offsets relative to the nullObj
eye.relOffX = 10; // offset 10 px left from nullObj center X ( 'd be -10 for left eye & +10 for right one or 0 for 1 eye only )
eye.relOffY = 0; // horizontally centered with nullObj ( that is itself centered )
// set the eye x's & y's relative to our nullObj's x & y
eye.x2 = eye.x3 = nullObj.x - eye.relOffX;
eye.x1 = eye.x4 = nullObj.x - (eye.relOffX+10);
eye.y3 = eye.y4 = nullObj.y - (eye.relOffY-10);
eye.y2 = eye.y1 = nullObj.y - (eye.relOffY+10);
// log those
console.log(eye);


// quick tweening
var easeLinear = function(t, b, c, d){ return c*t/d+b; }
var easeInQuad = function(t, b, c, d){ return c*(t/=d)*t+b; }


// quick debug
var eyeItem = document.querySelector('.eye');
eyeItem.style.left = eye.x1 + 'px';
eyeItem.style.top = eye.y1 + 'px';

var endOffsetX = 64;
var endOffsetY = 62;
var beginningX = eye.x1;
var beginningY = eye.y1;
var diffX = endOffsetX - beginningX;
var diffY = endOffsetY - beginningY;
var time = 0;
var duration = 150;
//var interval = 5;
var interval = 10;

var animItr = -1;
var animate = function(callback){
  //if(time < duration && beginningX < endOffsetX && beginningY < endOffsetY){
  //if(( time < duration && beginningX < endOffsetX ) || ( time < duration && beginningY < endOffsetY )){
  if(( time < duration && beginningX < endOffsetX ) ||
     ( time < duration && beginningX > endOffsetX ) ||
     ( time < duration && beginningY < endOffsetY ) ||
     ( time < duration && beginningY > endOffsetY ) ){
    //var px = easeLinear(time, beginningX, diffX, duration);
    //var py = easeLinear(time, beginningY, diffY, duration);
    var px = easeInQuad(time, beginningX, diffX, duration);
    var py = easeInQuad(time, beginningY, diffY, duration);
    // draw
    eyeItem.style.left = px + 'px';
    eyeItem.style.top = py + 'px';
    time += interval;
    animItr = setTimeout(function(){ animate(callback); }, interval);
  } else {
    //console.log('fk');
    time = 0;
    if(callback) callback();
  }
}
// try out
//animate();
//animate(function(){ console.log('animation ended !'); });
/*
animate(function(){
  console.log('animation 1 ended !');
  eye.x1 = endOffsetX;
  eye.y1 = endOffsetY;
  console.log(eye);
  //
  beginningX = eye.x1;
  beginningY = eye.y1;
  endOffsetX = 44+100;
  //endOffsetY = 22+100;
  diffX = endOffsetX - beginningX;
  diffY = endOffsetY - beginningY;
  //animate(function(){ console.log('animation 2 ended !'); });
  animate(function(){
    console.log('animation 2 ended !');
    eye.x1 = endOffsetX;
    eye.y1 = endOffsetY;
    console.log(eye);
    //
    beginningX = eye.x1;
    beginningY = eye.y1;
    //endOffsetX = 44+200; // ok
    endOffsetX = 44; // ok
    //endOffsetY = 22+100;
    diffX = endOffsetX - beginningX;
    diffY = endOffsetY - beginningY;
    animate(function(){ console.log('animation 3 ended !'); });
  });
});
*/




// ---- CODE TO BE ADDED TO AN ILLUSTRATOR PLUGIN TO GET EYES NULL OBJS FROM POLYGONS ----
function getRelPos(cx, cy, px, px){
  return { relX: cx-px, relY: cy-py};
}
var nullObjCenter = { x: 65.584, y: 37.57 }; // illustrator's nullObject center position
// absolute position of eyes polygons in the illustrator artboard
var neutral_eye1 = {
  x1: 30.33, y1: 23.5, x2: 62.67, y2: 23.5,
  x4: 30.33, y4: 55.83, x3:62.67, y3: 55.83,
};
var neutral_eye2 = {
  x1: 68.5, y1: 19.67, x2: 100.83, y2: 19.67,
  x4: 68.5, y4: 55.83, x3:100.83, y3: 55.83,
};
// relative positions that'll be computed from the nullObjCenter x & y coords
var neutral_relEye1 = {
  x1: 0, y1: 0, x2: 0, y2: 0,
  x4: 0, y4: 0, x3: 0, y3: 0,
};
var neutral_relEye2 = {
  x1: 0, y1: 0, x2: 0, y2: 0,
  x4: 0, y4: 0, x3: 0, y3: 0,
};
// compute both of the aboves
Object.keys(neutral_eye1).forEach(function(key){
  if(key.indexOf('x') !== -1 ) neutral_relEye1[key] = (nullObjCenter.x - neutral_eye1[key]).toFixed(1); // handle x1/2/3/4
  else if(key.indexOf('y') !== -1 ) neutral_relEye1[key] = (nullObjCenter.y - neutral_eye1[key]).toFixed(1); // // handle y1/2/3/4
});
Object.keys(neutral_eye2).forEach(function(key){
  if(key.indexOf('x') !== -1 ) neutral_relEye2[key] = (nullObjCenter.x - neutral_eye2[key]).toFixed(1); // handle x1/2/3/4
  else if(key.indexOf('y') !== -1 ) neutral_relEye2[key] = (nullObjCenter.y - neutral_eye2[key]).toFixed(1); // // handle y1/2/3/4
});




// -------- FACIAL EXPRESSION OBJ WIP --------
// -- FACIAL EXPRESSIONS
var fes = {
  "neutral":{"noc_x":65.58,"noc_y":37.75,"eyes":[{"x1":35.25,"y1":-18.08,"x2":2.92,"y2":-18.08,"x3":2.92,"y3":18.08,"x4":35.25,"y4":18.08},{"x1":-2.92,"y1":-18.08,"x2":-35.25,"y2":-18.08,"x3":-35.25,"y3":14.25,"x4":-2.92,"y4":14.25}]},
  "blink_high":{"noc_x":64.62,"noc_y":30.75,"eyes":[{"x1":0,"y1":-2.75,"x2":-60.38,"y2":-2.75,"x3":-60.38,"y3":2.75,"x4":0,"y4":2.75},{"x1":60.38,"y1":-2.75,"x2":0,"y2":-2.75,"x3":0,"y3":2.75,"x4":60.38,"y4":2.75}]},
  "happy":{"noc_x":65.58,"noc_y":27.85,"eyes":[{"x1":35.25,"y1":-4.35,"x2":2.92,"y2":-4.35,"x3":2.92,"y3":4.35,"x4":35.25,"y4":4.35},{"x1":-2.92,"y1":-4.35,"x2":-35.25,"y2":-4.35,"x3":-35.25,"y3":4.35,"x4":-2.92,"y4":4.35}]},
  "glee":{"noc_x":65.58,"noc_y":21.58,"eyes":[{"x1":37.25,"y1":-5.62,"x2":4.92,"y2":-3.62,"x3":4.92,"y3":5.08,"x4":37.25,"y4":3.08},{"x1":-4.92,"y1":-3.62,"x2":-37.25,"y2":-5.62,"x3":-37.25,"y3":3.62,"x4":-4.92,"y4":5.62}]},
  "blink_low":{"noc_x":64.62,"noc_y":46.75,"eyes":[{"x1":60.38,"y1":-2.75,"x2":0,"y2":-2.75,"x3":0,"y3":2.75,"x4":60.38,"y4":2.75},{"x1":0,"y1":-2.75,"x2":-60.38,"y2":-2.75,"x3":-60.38,"y3":2.75,"x4":0,"y4":2.75}]},
  "sad_lookingDown":{"noc_x":65.58,"noc_y":53.81,"eyes":[{"x1":35.25,"y1":-8.19,"x2":2.92,"y2":-8.19,"x3":2.92,"y3":8.19,"x4":35.25,"y4":2.19},{"x1":-2.92,"y1":-8.19,"x2":-35.25,"y2":-8.19,"x3":-35.25,"y3":2.19,"x4":-2.92,"y4":8.19}]},
  "sad_lookingArUser":{"noc_x":65.58,"noc_y":13.31,"eyes":[{"x1":35.25,"y1":-11.69,"x2":2.92,"y2":-11.69,"x3":2.92,"y3":11.69,"x4":35.25,"y4":5.69},{"x1":-2.92,"y1":-11.69,"x2":-35.25,"y2":-11.69,"x3":-35.25,"y3":5.69,"x4":-2.92,"y4":11.69}]},
  "worried":{"noc_x":65.58,"noc_y":41.56,"eyes":[{"x1":35.25,"y1":-16.19,"x2":2.92,"y2":-16.19,"x3":2.92,"y3":16.19,"x4":35.25,"y4":12.19},{"x1":-2.92,"y1":-16.19,"x2":-35.25,"y2":-16.19,"x3":-35.25,"y3":12.19,"x4":-2.92,"y4":16.19}]},
  "focused":{"noc_x":65.58,"noc_y":37.42,"eyes":[{"x1":35.25,"y1":-6.67,"x2":2.92,"y2":-6.67,"x3":2.92,"y3":6.67,"x4":35.25,"y4":6.67},{"x1":-2.92,"y1":-6.67,"x2":-35.25,"y2":-6.67,"x3":-35.25,"y3":6.67,"x4":-2.92,"y4":6.67}]},
  "annoyed":{"noc_x":65.58,"noc_y":31.88,"eyes":[{"x1":35.25,"y1":-0.12,"x2":2.92,"y2":-0.12,"x3":2.92,"y3":4.25,"x4":35.25,"y4":4.25},{"x1":-2.92,"y1":-4.25,"x2":-35.25,"y2":-4.25,"x3":-35.25,"y3":3.12,"x4":-2.92,"y4":3.12}]},
  "surprised":{"noc_x":66.08,"noc_y":36.06,"eyes":[{"x1":42.75,"y1":-20.69,"x2":0.42,"y2":-24.69,"x3":0.42,"y3":24.69,"x4":42.75,"y4":18.69},{"x1":-1.42,"y1":-24.69,"x2":-42.75,"y2":-20.69,"x3":-42.75,"y3":18.69,"x4":-1.42,"y4":24.69}]},
  "skeptic":{"noc_x":65.58,"noc_y":37.56,"eyes":[{"x1":35.25,"y1":-16.19,"x2":2.92,"y2":-16.19,"x3":2.92,"y3":3.19,"x4":35.25,"y4":7.19},{"x1":-2.92,"y1":-16.19,"x2":-35.25,"y2":-16.19,"x3":-35.25,"y3":16.19,"x4":-2.92,"y4":16.19}]},
  "frustrated":{"noc_x":68.58,"noc_y":49.56,"eyes":[{"x1":35.25,"y1":-4.19,"x2":2.92,"y2":-4.19,"x3":2.92,"y3":4.19,"x4":35.25,"y4":4.19},{"x1":-2.92,"y1":-4.19,"x2":-35.25,"y2":-4.19,"x3":-35.25,"y3":4.19,"x4":-2.92,"y4":4.19}]},
  "unimpressed":{"noc_x":66.58,"noc_y":48.06,"eyes":[{"x1":35.25,"y1":-5.69,"x2":2.92,"y2":-5.69,"x3":2.92,"y3":5.69,"x4":35.25,"y4":5.69},{"x1":-2.92,"y1":-5.69,"x2":-35.25,"y2":-5.69,"x3":-35.25,"y3":2.69,"x4":-2.92,"y4":2.69}]},
  "sleepyEyes":{"noc_x":62.67,"noc_y":46.03,"eyes":[{"x1":35.88,"y1":-7.83,"x2":3.62,"y2":-5.6,"x3":4.12,"y3":1.64,"x4":36.38,"y4":-0.6},{"x1":-0.69,"y1":-7.67,"x2":-34.07,"y2":-11.65,"x3":-36.38,"y3":7.67,"x4":-2.99,"y4":11.65}]},
  "suspicious":{"noc_x":65.58,"noc_y":36.25,"eyes":[{"x1":38.25,"y1":-3.5,"x2":2.92,"y2":-4.5,"x3":2.92,"y3":3.5,"x4":38.25,"y4":4.5},{"x1":-2.92,"y1":-4.5,"x2":-38.25,"y2":-3.5,"x3":-38.25,"y3":4.5,"x4":-2.92,"y4":3.5}]},
  "squint":{"noc_x":65.58,"noc_y":36.25,"eyes":[{"x1":37.25,"y1":-3.5,"x2":2.92,"y2":-4.5,"x3":2.92,"y3":4.5,"x4":37.25,"y4":4.5},{"x1":-2.92,"y1":-4.5,"x2":-37.25,"y2":-3.5,"x3":-37.25,"y3":4.5,"x4":-2.92,"y4":4.5}]},
  "angry":{"noc_x":65.58,"noc_y":45.56,"eyes":[{"x1":35.25,"y1":-8.19,"x2":2.92,"y2":-8.19,"x3":2.92,"y3":4.19,"x4":35.25,"y4":8.19},{"x1":-2.92,"y1":-8.19,"x2":-35.25,"y2":-8.19,"x3":-35.25,"y3":8.19,"x4":-2.92,"y4":4.19}]},
  "furious":{"noc_x":65.58,"noc_y":35.06,"eyes":[{"x1":35.25,"y1":-11.69,"x2":2.92,"y2":-11.69,"x3":2.92,"y3":3.69,"x4":35.25,"y4":11.69},{"x1":-2.92,"y1":-11.69,"x2":-35.25,"y2":-11.69,"x3":-35.25,"y3":11.69,"x4":-2.92,"y4":3.69}]},
  "scared":{"noc_x":65.58,"noc_y":21.56,"eyes":[{"x1":35.25,"y1":-16.19,"x2":2.92,"y2":-16.19,"x3":2.92,"y3":16.19,"x4":35.25,"y4":12.19},{"x1":-2.92,"y1":-16.19,"x2":-35.25,"y2":-16.19,"x3":-35.25,"y3":12.19,"x4":-2.92,"y4":16.19}]},
  "awe":{"noc_x":66.08,"noc_y":31.56,"eyes":[{"x1":41.75,"y1":-21.19,"x2":2.42,"y2":-25.19,"x3":2.42,"y3":25.19,"x4":41.75,"y4":19.19},{"x1":-3.42,"y1":-25.19,"x2":-41.75,"y2":-21.19,"x3":-41.75,"y3":19.19,"x4":-3.42,"y4":25.19}]}
};
// -- ANIM HELPER
var animStep = function(time, tweenFcn, beginningX, beginningY, diffX, diffY, duration, updateCallback, endCallback){
  //if(time < duration && beginningX < endOffsetX && beginningY < endOffsetY){
  //if(( time < duration && beginningX < endOffsetX ) || ( time < duration && beginningY < endOffsetY )){
  if(( time < duration && beginningX < endOffsetX ) ||
     ( time < duration && beginningX > endOffsetX ) ||
     ( time < duration && beginningY < endOffsetY ) ||
     ( time < duration && beginningY > endOffsetY ) ){
    //var px = easeLinear(time, beginningX, diffX, duration);
    //var py = easeLinear(time, beginningY, diffY, duration);
    var px = tweenFcn(time, beginningX, diffX, duration);
    var py = tweenFcn(time, beginningY, diffY, duration);
    // draw
    updateCallback(px, py); 
    time += interval;
    //animItr = setTimeout(function(){ animate(callback); }, interval);
    animItr = setTimeout(function(){ animStep(time, tweenFcn, beginningX, beginningY, diffX, diffY, duration, updateCallback, endCallback); }, interval);
  } else {
    //console.log('fk');
    time = 0;
    if(endCallback) endCallback();
  }
}
// -- FACIAL EXPRESSION HELPER
var fe = {
  ce: '', // current expression
  noc_x: 0,
  noc_y: 0,
  eyes: [],
  that: this,
  set: function(expressionName){
    if(fes[expressionName]){
      var exprObj = fes[expressionName];
      this.ce = expressionName;
      this.noc_x = 128- exprObj.noc_x;
      this.noc_y = 64- exprObj.noc_y;
      this.eyes = [];
      var exprEyesArr = fes[expressionName].eyes;
      for(var i=0; i< exprEyesArr.length; i++){
        this.eyes.push({
          x1: exprEyesArr[i].x1, y1: exprEyesArr[i].y1, x2: exprEyesArr[i].x2, y2: exprEyesArr[i].y2,
          x4: exprEyesArr[i].x4, y4: exprEyesArr[i].y4, x3: exprEyesArr[i].x3, y3: exprEyesArr[i].y3
        });
      }
    } else { console.log('expression not found'); }
  },
  update: function(duration, expression, blink, callback){
    // draw it directly if duration == 0, else animate the morphing
    if(duration === 0){ // draw directly
      g.clear();
      //console.log(this.eyes);
      // draw the eyes in correct position
      for(var i=0; i< this.eyes.length; i++){
        g.fillPoly([this.noc_x+this.eyes[i].x1, this.noc_y+this.eyes[i].y1, this.noc_x+this.eyes[i].x2, this.noc_y+this.eyes[i].y2,
                    this.noc_x+this.eyes[i].x3, this.noc_y+this.eyes[i].y3, this.noc_x+this.eyes[i].x4, this.noc_y+this.eyes[i].y4]);
      }
      g.flip();
    } else { // animate the morphing for specified duration
      console.log('currently not supporting animation between expressions ..');
      // handle only transitions to other nullObject center for now
      var endOffsetX = fes[expression].noc_x; console.log('fes[expression].noc_x:' + fes[expression].noc_x);
      var endOffsetY = fes[expression].noc_y; console.log('fes[expression].noc_y:' + fes[expression].noc_y);
      var beginningX = fes[this.ce].noc_x; console.log('fes[this.ce].noc_x:' + fes[this.ce].noc_x);
      var beginningY = fes[this.ce].noc_y; console.log('fes[this.ce].noc_y:' + fes[this.ce].noc_y);
      var diffX = endOffsetX - beginningX; console.log('diffY:' + diffX);
      var diffY = endOffsetY - beginningY; console.log('diffY:' + diffY);
      var time = 0;
      var duration = 150;
      var interval = 10;
      animStep(time, easeInQuad, beginningX, beginningY, diffX, diffY, duration, function(px, py){
        this.noc_x = px;
        this.noc_y = py;
        g.clear();
        // draw the eyes in correct position
        //for(var i=0; i< this.eyes.length; i++){
        /*
        for(var i=0; i< that.eyes.length; i++){
          g.fillPoly([this.noc_x+this.eyes[i].x1, this.noc_y+this.eyes[i].y1, this.noc_x+this.eyes[i].x2, this.noc_y+this.eyes[i].y2,
                    this.noc_x+this.eyes[i].x3, this.noc_y+this.eyes[i].y3, this.noc_x+this.eyes[i].x4, this.noc_y+this.eyes[i].y4]);
        }
        */
        /*
        for(var i=0; i< fe.length; i++){
          g.fillPoly([fe.noc_x+fe.eyes[i].x1, fe.noc_y+fe.eyes[i].y1, fe.noc_x+fe.eyes[i].x2, fe.noc_y+fe.eyes[i].y2,
                    fe.noc_x+fe.eyes[i].x3, fe.noc_y+fe.eyes[i].y3, fe.noc_x+fe.eyes[i].x4, fe.noc_y+fe.eyes[i].y4]);
        }
        */
        fe.update(0);
        g.flip(); // for Espruino
      }, function(){
        console.log('transition to expression: '+expression+' ended !');
        this.ce = expression;
        if(callback) callback();
      });
    }
  },
  // move the random nullObject center to get an expression-agnostic x & y offset movement
  randMoveNoc: function(min, max, durationMin, durationMax, repeat, callback){},
  blink: function(fromBaseLine){}, // close then open both eyes, fromBaseLine bool to offset the blink
  closeEye: function(){}, // close the eye(s) whose idxs are passed as parameters or all if no args
  openEye: function(){}, // // close the eye(s) whose idxs are passed as parameters or all if no args
};
var feIdx = 0; // to quickly cycle through all facial expressions ( debug )
var fesKeys = Object.keys(fes);
var nextFe = function(){
  if(feIdx < fesKeys.length-1) feIdx++;
  else feIdx = 0;
  fe.set( fesKeys[feIdx] );
  fe.update(0);
};
var prevFe = function(){
  if(feIdx-1 >= 0) feIdx--;
  else feIdx = fesKeys.length-1;
  fe.set( fesKeys[feIdx] );
  fe.update(0);
};

// ---- CANVAS PREVIEW OF OLED SCREEN DISPLAY ----
myCanvas.height = 64;
myCanvas.width = 128;

// -- adding Espruino's graphics methods to canvas to ease testing
/*
CanvasRenderingContext2D.prototype.fillPolygon = function (pointsArray) {
  if (pointsArray.length <= 0) return;
  this.moveTo(pointsArray[0][0], pointsArray[0][1]);
  for (var i = 0; i < pointsArray.length; i++) {
    this.lineTo(pointsArray[i][0], pointsArray[i][1]);
  }
  if (strokeColor != null && strokeColor != undefined)
    this.strokeStyle = strokeColor;

  if (fillColor != null && fillColor != undefined) {
    this.fillStyle = fillColor;
    this.fill();
  }
}
*/
// usage:
//var polygonPoints = [[10,100],[20,75],[50,100],[100,100],[10,100]];
//context.fillPolygon(polygonPoints, '#F00','#000');
// my mod for Espruino
//CanvasRenderingContext2D.prototype.fillPoly = function () {
  //var pointsArray = arguments;
CanvasRenderingContext2D.prototype.fillPoly = function (pointsArray) {
  if (pointsArray.length <= 0) return;
  this.moveTo(pointsArray[0], pointsArray[1]);
  this.beginPath();
  for (var i = 0; i < pointsArray.length; i+=2) {
    this.lineTo(pointsArray[i], pointsArray[i+1]);
  }
  this.lineTo(pointsArray[0], pointsArray[1]); // closed shape -> re-use 1st coord
  this.fill();
}
CanvasRenderingContext2D.prototype.setColor = function(color){
  this.fillStyle = color;
  this.strokeStyle = color;
}
CanvasRenderingContext2D.prototype.clear = function(){
  var currFillStyle = this.fillStyle; // bckp
  this.fillStyle = 'black';
  this.fillRect(0, 0, this.canvas.width, this.canvas.height);
  this.fillStyle = currFillStyle; // restore bckp
  //this.clearRect(0, 0, this.canvas.width, this.canvas.height);
  //console.log('canvas cleared');
}
CanvasRenderingContext2D.prototype.flip = function(){
  // TODO: maybe some sort of 'reveal stuff already drawn' ? for now, just have a log ;)
  //console.log('canvas flipped');
}
CanvasRenderingContext2D.prototype.setRotation = function(rotation){
  // 0 = 0°
  // 1 = 90°
  // 2 = 180°
  // 3 = 270°
  if(rotation === 0){ this.setTransform(1, 0, 0, 1, 0, 0); }
  else if(rotation === 2){
    this.translate(this.canvas.width, this.canvas.height);
    this.rotate(rotation*90 * Math.PI / 180);
    //this.translate(this.canvas.width, 0);
    //this.scale(-1, 1); // mirroring as well
  }
  //console.log('canvas rotated');
}

/*
var ctx = myCanvas.getContext('2d');
ctx.fillStyle = '#f00';
ctx.beginPath();
ctx.moveTo(0, 0);
ctx.lineTo(100,50);
ctx.lineTo(50, 100);
ctx.lineTo(0, 90);
ctx.closePath();
ctx.fill();
*/
var g = myCanvas.getContext('2d');
g.setRotation(2);
g.setColor('white');
//g.fillPoly([10, 10, 20, 10, 20, 20, 10, 20]); // square: ok
//g.flip();
//g.clear();
//g.fillPoly([eye.x1, eye.y1, eye.x2, eye.y2, eye.x3, eye.y3, eye.x4, eye.y4]); // ok: polygon
//g.flip();
// hopefully, no error ? ( code below is from Espruino wip implm )
fe.set('happy'); // ok
fe.update(0);

// cycle between available facial expressions ( don't transition between thse for now )
var tout = -1;
//tout = setInterval(function(){ nextFe(); console.log('current expression:' + feIdx + ' --> ' + fe.ce)}, 1000);
//fe.set('happy'); // ok
//fe.update(3, 'worried', false, function(){ console.log('transitionning to nxt noc done !') });


```
