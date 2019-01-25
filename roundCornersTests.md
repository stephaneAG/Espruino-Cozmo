JSBIN
https://jsbin.com/kovihocugi/edit?html,css,js,console,output

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
  <!-- <canvas id="memcanvas"></canvas> -->
  <canvas id="canvas"></canvas>
</body>
</html>
```
CSS
```css
#memcanvas {
  border: 1px solid blue;
}
#canvas {
  border: 1px solid red;
}
```
JS
```javascript
// R: goal is to be able to draw filled or contoured rounded polygons using the following method:
// https://github.com/espruino/Espruino/blob/master/libs/graphics/graphics.c#L385

// rounded corners poly ( thx to https://riptutorial.com/html5-canvas/example/18766/render-a-rounded-polygon- )
var roundedPoly = function(ctx, points,radius){
    var i, x, y, len, p1, p2, p3, v1, v2, sinA, sinA90, radDirection, drawDirection, angle, halfAngle, cRadius, lenOut;
    var asVec = function (p, pp, v) { // convert points to a line with len and normalised
        v.x = pp.x - p.x; // x,y as vec
        v.y = pp.y - p.y;
        v.len = Math.sqrt(v.x * v.x + v.y * v.y); // length of vec
        v.nx = v.x / v.len; // normalised
        v.ny = v.y / v.len;
        v.ang = Math.atan2(v.ny, v.nx); // direction of vec
    }
    v1 = {};
    v2 = {};
    len = points.length;                         // number points
    p1 = points[len - 1];                        // start at end of path
    for (i = 0; i < len; i++) {                  // do each corner
        p2 = points[(i) % len];                  // the corner point that is being rounded
        p3 = points[(i + 1) % len];
        // get the corner as vectors out away from corner
        asVec(p2, p1, v1);                       // vec back from corner point
        asVec(p2, p3, v2);                       // vec forward from corner point
        // get corners cross product (asin of angle)
        sinA = v1.nx * v2.ny - v1.ny * v2.nx;    // cross product
        // get cross product of first line and perpendicular second line
        sinA90 = v1.nx * v2.nx - v1.ny * -v2.ny; // cross product to normal of line 2
        angle = Math.asin(sinA);                 // get the angle
        radDirection = 1;                        // may need to reverse the radius
        drawDirection = false;                   // may need to draw the arc anticlockwise
        // find the correct quadrant for circle center
        if (sinA90 < 0) {
            if (angle < 0) {
                angle = Math.PI + angle; // add 180 to move us to the 3 quadrant
            } else {
                angle = Math.PI - angle; // move back into the 2nd quadrant
                radDirection = -1;
                drawDirection = true;
            }
        } else {
            if (angle > 0) {
                radDirection = -1;
                drawDirection = true;
            }
        }
        halfAngle = angle / 2;
        // get distance from corner to point where round corner touches line
        lenOut = Math.abs(Math.cos(halfAngle) * radius / Math.sin(halfAngle));
        if (lenOut > Math.min(v1.len / 2, v2.len / 2)) { // fix if longer than half line length
            lenOut = Math.min(v1.len / 2, v2.len / 2);
            // ajust the radius of corner rounding to fit
            cRadius = Math.abs(lenOut * Math.sin(halfAngle) / Math.cos(halfAngle));
        } else {
            cRadius = radius;
        }
        x = p2.x + v2.nx * lenOut; // move out from corner along second line to point where rounded circle touches
        y = p2.y + v2.ny * lenOut;
        x += -v2.ny * cRadius * radDirection; // move away from line to circle center
        y += v2.nx * cRadius * radDirection;
        // x,y is the rounded corner circle center
        ctx.arc(x, y, cRadius, v1.ang + Math.PI / 2 * radDirection, v2.ang - Math.PI / 2 * radDirection, drawDirection); // draw the arc clockwise
        p1 = p2;
        p2 = p3;
    }
    ctx.closePath();
}
// typical usage
/*
var triangle = [
    { x: 200, y : 50 },
    { x: 300, y : 200 },
    { x: 100, y : 200 }
];
var cornerRadius = 30;
g.lineWidth = 4;
g.fillStyle = "Green";
g.strokeStyle = "black";
g.beginPath(); // start a new path
roundedPoly(g, triangle, cornerRadius);
g.fill();
g.stroke();
*/


// Flo's baking hint
var BAKE_STEP_COUNT = 32;
var BAKE_STEP = (BAKE_STEP_COUNT > 0) ? 1.0 / BAKE_STEP_COUNT : 1.0;
var t = 0.0;
var samples = [];
/*
for (int i = 0; i < BAKE_STEP_COUNT; ++i){
  var samplePosition = { x: 0, y: 0}; //Vector2 samplePosition = new Vector2();
  BezierCubic(t, 
              k.Position,
              k.Position + k.TangentCurrent,
              kNext.Position + kNext.TangentPrev,
              kNext.Position,
              ref samplePosition
             );
  Samples.Add(new Sample(samplePosition, 0.0f));
  t += BAKE_STEP;
}
*/

// wip bezierCurveTo() & quadraticCurveTo() -> thx to Flo & wikipedia ;p
function bezierCurve(t, p0x, p0y, p1x, p1y, p2x, p2y, p3x, p3y){
  var t2 = t * t;
  var t3 = t2 * t;
  var oneMinT = 1 - t;
  var oneMinT2 = oneMinT * oneMinT;
  var oneMinT3 = oneMinT2 * oneMinT;

  return {
    x: p0x * oneMinT3 + 3 * p1x * t * oneMinT2 + 3 * p2x * t2 * oneMinT + p3x * t3,
    y: p0y * oneMinT3 + 3 * p1y * t * oneMinT2 + 3 * p2y * t2 * oneMinT + p3y * t3
  };
}
// is it good ? -> to test !
function quadraticCurve(t, p0x, p0y, p1x, p1y, p2x, p2y){
  var t2 = t * t;
  var oneMinT = 1 - t;
  var oneMinT2 = oneMinT * oneMinT;

  return {
    x: p0x * oneMinT2 + 2 * p1x * t * oneMinT + p2x *t2,
    y: p0y * oneMinT2 + 2 * p1y * t * oneMinT + p2y *t2
  };
}
// also good ? -> to test !
function linearCurve(t, p0x, p0y, p1x, p1y){
  var t2 = t * t;
  var oneMinT = 1 - t;
  var oneMinT2 = oneMinT * oneMinT;

  return {
    x: p0x * oneMinT + p1x,
    y: p0y * oneMinT2 + 2 * p1y * t * oneMinT + p2y *t2
  };
}


var Graphics = function(width, height, callback){
  //this.cnvsBuff = document.querySelector('#memcanvas'); //document.createElement('canvas');//new OffscreenCanvas(width, height);
  this.cnvsBuff = document.createElement('canvas');
  this.cnvsBuff.width = width;
  this.cnvsBuff.height = height;
  this.cnvsBuffCtx = this.cnvsBuff.getContext('2d');
  this.cnvs = document.querySelector('#canvas'); //document.createElement('canvas');
  this.cnvs.width = width;
  this.cnvs.height = height;
  this.cnvsCtx = this.cnvs.getContext('2d');
  var _self = this;

  // Espruino-like API
  //OffscreenCanvasRenderingContext2D.prototype.fillPoly = function (pointsArray) {
  CanvasRenderingContext2D.prototype.setPixel = function (x, y) {
    this.fillRect( x, y, 1, 1 );
  }
  
  //OffscreenCanvasRenderingContext2D.prototype.fillPoly = function (pointsArray) {
  CanvasRenderingContext2D.prototype.drawPoly = function (pointsArray, closed) {
    if (pointsArray.length <= 0) return;
    this.moveTo(pointsArray[0], pointsArray[1]);
    this.beginPath();
    for (var i = 0; i < pointsArray.length; i+=2) {
      this.lineTo(pointsArray[i], pointsArray[i+1]);
    }
    if(arguments.length > 1 && closed) 
      this.lineTo(pointsArray[0], pointsArray[1]); // closed shape -> re-use 1st coord
    this.stroke();
  }
  //OffscreenCanvasRenderingContext2D.prototype.fillPoly = function (pointsArray) {
  CanvasRenderingContext2D.prototype.fillPoly = function (pointsArray) {
    if (pointsArray.length <= 0) return;
    this.moveTo(pointsArray[0], pointsArray[1]);
    this.beginPath();
    for (var i = 0; i < pointsArray.length; i+=2) {
      this.lineTo(pointsArray[i], pointsArray[i+1]);
    }
    this.lineTo(pointsArray[0], pointsArray[1]); // closed shape -> re-use 1st coord
    this.fill();
    this.closePath();
  }
  //OffscreenCanvasRenderingContext2D.prototype.setColor = function(color){
  CanvasRenderingContext2D.prototype.setColor = function(color){
    this.fillStyle = color;
    this.strokeStyle = color;
  }
  //OffscreenCanvasRenderingContext2D.prototype.clear = function(){
  CanvasRenderingContext2D.prototype.clear = function(){
    var currFillStyle = this.fillStyle; // bckp
    this.fillStyle = 'black';
    this.fillRect(0, 0, this.canvas.width, this.canvas.height);
    this.fillStyle = currFillStyle; // restore bckp
    //this.clearRect(0, 0, this.canvas.width, this.canvas.height);
    //console.log('canvas cleared');
  }
  //OffscreenCanvasRenderingContext2D.prototype.flip = function(){
  CanvasRenderingContext2D.prototype.flip = function(){
    // TODO: maybe some sort of 'reveal stuff already drawn' ? for now, just have a log ;)
    //this.cnvsCtx.putImageData(this.cnvsBuffCtx.getImageData(0,0,this.cnvs.width,this.cnvs.height), 0, 0);
    //var buffImg = this.cnvsBuffCtx.getImageData(0,0,this.cnvs.width,this.cnvs.height);
    //var buffImg = this.cnvsBuffCtx.getImageData(0, 0, 150, 150);
    //var buffImg = _self.cnvsBuffCtx.getImageData(0, 0, 150, 150);
    var buffImg = _self.cnvsBuffCtx.getImageData(0, 0, _self.cnvs.width, _self.cnvs.height);
    //this.cnvsCtx.putImageData(buffImg, 0, 0);
    _self.cnvsCtx.putImageData(buffImg, 0, 0);
    //console.log('canvas flipped');
  }
  //OffscreenCanvasRenderingContext2D.prototype.setRotation = function(rotation){
  CanvasRenderingContext2D.prototype.setRotation = function(rotation){
    // 0 = 0°
    // 1 = 90°
    // 2 = 180°
    // 3 = 270°
    if(rotation === 0){ this.setTransform(1, 0, 0, 1, 0, 0); }
    else if(rotation === 1){
      this.scale(1, 1);
      this.translate(this.canvas.width, this.canvas.height);
      this.rotate(90 * Math.PI / 180);
      this.translate(-this.canvas.width, 0);
      //this.scale(-1, 1); // mirroring as well
    } else if(rotation === 2){
      this.scale(1, 1);
      this.translate(this.canvas.width, this.canvas.height);
      this.rotate(rotation*90 * Math.PI / 180);
      //this.translate(this.canvas.width, 0);
      //this.scale(-1, 1); // mirroring as well
    } else if(rotation === 3){
      this.scale(1, 1);
      /*
      this.translate(this.canvas.width, this.canvas.height);
      this.rotate(-90 * Math.PI / 180);
      this.translate(0, -this.canvas.width);
      */
      //this.translate(0, 0);
      this.translate(this.canvas.width, this.canvas.height);
      this.scale(-1, -1); // mirroring as well
    }
    //console.log('canvas rotated');
  }

  //if(callback) callback();
  return this.cnvsBuffCtx;
};

// simulate OLED & draw stuff
//var g = new Graphics(128, 64);
//var g = new Graphics(100, 100);
var g = new Graphics(300, 300);
g.setRotation(0);
//g.clear();
g.setColor('red');
g.fillPoly([10, 10, 20, 10, 20, 20, 10, 20]);
g.flip();

g.setRotation(2);
g.setColor('blue');
//g.drawPoly([10, 10, 20, 10, 20, 20, 10, 20]);
g.drawPoly([10, 10, 20, 10, 20, 20, 10, 20], true);
g.flip();

g.setRotation(1);
g.setColor('green');
g.fillPoly([10, 10, 20, 10, 20, 20, 10, 20]);
g.flip();

g.setRotation(3);
g.setColor('yellow');
g.fillPoly([10, 10, 20, 10, 20, 20, 10, 20]);
g.flip();

g.setRotation(0);
g.setColor('white');
g.setPixel(50, 50);
g.flip();

// quicky try of our curves fcns
// - via canvas fcn
g.beginPath(); // start a new path
g.setColor('red');
g.moveTo(20, 20);
g.quadraticCurveTo(20, 100, 200, 20);
g.stroke();
var p0 = { x: 20, y: 20+10};
var p1 = { x: 20, y: 100+10};
var p2 = { x: 200, y: 20+10};
// - via Flo's helper :)
var time = 0;
//var stepping = 0.001; // ok ?
//var stepping = 0.01; // shorter ?
var stepping = 0.005; // seems the nicest
//var stepping = 0.1; // ok ?
//var stepping = 0.125;
var pathPts = [];
for(time = 0; time <= 1; time+= stepping){
  //pathPts.push( quadraticCurve(time, p0x, p0y, p1x, p1y, p2x, p2y) );
  var pos = quadraticCurve(time, p0.x, p0.y, p1.x, p1.y, p2.x, p2.y);
  pathPts.push(pos.x, pos.y);
}
g.setColor('turquoise');
g.drawPoly(pathPts, false);


// trying quick rounded poly
/**/
var triangle = [
  { x: 200, y : 50 },
  { x: 300, y : 200 },
  { x: 100, y : 200 }
];
var square = [
  { x: 100, y : 100 },
  { x: 200, y : 100 },
  { x: 200, y : 200 },
  { x: 100, y : 200 }
];
//var cornerRadius = 30;
//var cornerRadius = 20;
var cornerRadius = 40;
//g.lineWidth = 4;
g.setColor('purple');
g.fillStyle = "Transparent"; //"Green";
//g.strokeStyle = "white";
g.beginPath(); // start a new path
//roundedPoly(g, triangle, cornerRadius);
roundedPoly(g, square, cornerRadius);
g.fill();
g.stroke();
g.flip();
/**/



// trying quick svg outta Illustrator ( save as > svg > show )
/*
<path fill="#ED1845" d="M11.084,10.833c0.925-5.704,2-16.167,19.25-2.5c8.148,6.455,4.25,10.834,0,13.167
	c-2.361,1.296-13.064,5.343-17.75,2.667C8.838,22.026,10.64,13.574,11.084,10.833z"></path>
*/
/*
R: https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths
M 11.084, 10.833 --> moveTo(x, y)
c 0.925-5.704, 2-16.167, 19.25-2.5 --> Quadratic x1 y1, x y (or q dx1 dy1, dx dy) ?
c 8.148, 6.455, 4.25, 10.834, 0, 13.167 --> Cubic x1 y1, x2 y2, x y (or c dx1 dy1, dx2 dy2, dx dy)
c -2.361, 1.296-13.064, 5.343-17.75, 2.667 --> Quadratic x1 y1, x y (or q dx1 dy1, dx dy) ?
C 8.838, 22.026, 10.64, 13.574, 11.084, 10.833 --> Cubic x1 y1, x2 y2, x y (or c dx1 dy1, dx2 dy2, dx dy)
z --> back to the start
*/
// below one works, but appears in upper-left corner
var p = new Path2D('M11.084,10.833c0.925-5.704,2-16.167,19.25-2.5c8.148,6.455,4.25,10.834,0,13.167c-2.361,1.296-13.064,5.343-17.75,2.667C8.838,22.026,10.64,13.574,11.084,10.833z');
// other test (setting path after moving the Path2D )
//var p2 = new Path2D();
//p2.moveTo(150, 150);
//p.addPath('M11.084,10.833c0.925-5.704,2-16.167,19.25-2.5c8.148,6.455,4.25,10.834,0,13.167c-2.361,1.296-13.064,5.343-17.75,2.667C8.838,22.026,10.64,13.574,11.084,10.833z');
//p2.addPath(p);
g.stroke(p);
g.flip();

/* // working shape ( rounded rectangle )
var p3 = new Path2D('M128.411,169.142c0,4.971,4.029,9,9,9h14.333c4.971,0,9-4.029,9-9v-14.334c0-4.971-4.029-9-9-9h-14.333c-4.971,0-9,4.029-9,9V169.142z');
g.stroke(p3);
g.fillStyle = 'green';
//g.fill(p3);
*/
g.flip();

// additional quick try of curve fcns
// to draw a 'round rectangle' without ABSOLUTE points coords from Illustrator
var eye = {
           x1:     128.41,
           y1:    -169.14,
           x1in:   128.41,
           y1in:  -169.14,
           x1out:  128.41,
           y1out: -174.11,
           
           x2:     137.41,
           y2:    -178.14,
           x2in:   132.44,
           y2in:  -178.14,
           x2out:  137.41,
           y2out: -178.14,
  
           x3:     151.74,
           y3:    -178.14,
           x3in:   151.74,
           y3in:  -178.14,
           x3out:  156.71,
           y3out: -178.14,
  
           x4:     160.74,
           y4:    -169.14,
           x4in:   160.74,
           y4in:  -174.11,
           x4out:  160.74,
           y4out: -169.14,
  
           x5:     160.74,
           y5:    -154.81,
           x5in:   160.74,
           y5in:  -154.81,
           x5out:  160.74,
           y5out: -149.84,
  
           x6:     151.74,
           y6:    -145.81,
           x6in:   156.71,
           y6in:  -145.81,
           x6out:  151.74,
           y6out: -145.81,
  
           x7:     137.41,
           y7:    -145.81,
           x7in:   137.41,
           y7in:  -145.81,
           x7out:  132.44,
           y7out: -145.81,
  
           x8:     128.41,
           y8:    -154.81,
           x8in:   128.41,
           y8in:  -149.84,
           x8out:  128.41,
           y8out: -154.81
          };

g.beginPath(); // start a new path
g.setColor('red');
//g.moveTo(10, 10);
//var yOffset = 100 + g.canvas.width/2;
var yOffset = 174 + g.canvas.width/2;
g.moveTo(eye.x1, eye.y1+yOffset);
//g.lineTo(100, 100);
//g.moveTo(eye.x1, eye.y1+yOffset);
//g.quadraticCurveTo(eye.x1in, eye.y1in+yOffset, eye.x1out, eye.y1out+yOffset);
//g.quadraticCurveTo(eye.x1in, eye.y1in+yOffset, eye.x1out, eye.y1out+yOffset);
//g.bezierCurveTo(eye.x1in, eye.y1in+yOffset, eye.x1out, eye.y1out+yOffset, eye.x2, eye.y2+yOffset);
//g.moveTo(eye.x2, eye.y2+yOffset);
//g.bezierCurveTo(eye.x2in, eye.y2in+yOffset, eye.x2out, eye.y2out+yOffset, eye.x3, eye.y3+yOffset);
//g.bezierCurveTo(eye.x3in, eye.y3in+yOffset, eye.x3out, eye.y3out+yOffset, eye.x4, eye.y4+yOffset);
//g.bezierCurveTo(eye.x4in, eye.y4in+yOffset, eye.x4out, eye.y4out+yOffset, eye.x5, eye.y5+yOffset);
// ..
/* // working rounded rect
for(var i=1; i < 8; i++){
  g.bezierCurveTo(eye['x'+i+'in'],
                  eye['y'+i+'in']+yOffset,
                  eye['x'+i+'out'],
                  eye['y'+i+'out']+yOffset,
                  eye['x'+(i+1)],
                  eye['y'+(i+1)]+yOffset);
}
g.bezierCurveTo(eye.x8in, eye.y8in+yOffset, eye.x8out, eye.y8out+yOffset, eye.x1, eye.y1+yOffset);
g.stroke();
g.flip();
*/

// above result isclose to Path2D() using SVG path exported from Illu
// - via Flo's helper :)
var time = 0;
var stepping = 0.005; // seems the nicest
var pathPts = [];
g.setColor('black');
g.beginPath();
for(var i=1; i < 8; i++){
  time=0;
  //pathPts = [];
  for(time = 0; time <= 1; time+= stepping){
    //var pos = quadraticCurve(time, p0.x, p0.y, p1.x, p1.y, p2.x, p2.y);
    //var pos = bezierCurve(t, p0x, p0y, p1x, p1y, p2x, p2y, p3x, p3y);
    var pos = bezierCurve(time, eye['x'+i], eye['y'+i] +yOffset,
                             eye['x'+i+'in'], eye['y'+i+'in'] +yOffset,
                             eye['x'+i+'out'], eye['y'+i+'out'] +yOffset,
                             eye['x'+(i+1)], eye['y'+(i+1)] +yOffset);
    pathPts.push(pos.x, pos.y);
  }
  //g.drawPoly(pathPts, false);
  //g.fillStyle = 'green';
  //g.fill();
  //g.stroke();
  //g.fillPoly(pathPts);
  //console.log(pathPts);
}
g.moveTo(pathPts[0], pathPts[1]); 
for (var i = 0; i < pathPts.length; i+=2) {
  g.lineTo(pathPts[i], pathPts[i+1]);
}
g.lineTo(pathPts[0], pathPts[1]); // closed shape -> re-use 1st coord
g.fill();
//g.stroke();
g.closePath();
g.flip();

```
