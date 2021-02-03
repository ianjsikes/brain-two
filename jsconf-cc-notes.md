# jsconf cc notes

## Intro

So let's see what we've got here. We have a basic React app, created using create-react-app.

(Show dependencies)
I've added
- react-three-fiber and three
- @react-three/drei, a library full of neat utilities for r3f
- react-spring, an animation library that works well with r3f

(Show styles.css)
Here we have some very minimal css to make the React app fill the whole screen. We could change the background color here if we want.

(Show index.js)
Right now we are just rendering a header.

## Step 1

### Add canvas

Let's get something exciting going. First we are going to render a Canvas
from react-three-fiber. This sets up the HTML canvas element and will
wrap around all of our entire 3D scene.

So we'll replace the H1 with the Canvas. We don't see anything yet,
which makes sense. Let's add a bit of configuration real quick.
- colorManagement just makes the colors look nicer
- shadowMap let's us have nice shadows
- let's set the initial position of the camera

### Add Sphere

Now to make something show up. Let's just make a sphere to start. Every
object we make is going to follow the same basic pattern. First, we make
a mesh component. We can pass additional props to this like position,
scale, etc. For now we will stick with the defaults.

Then, as children to the mesh, we need a _geometry_ and a _material_.
We're gonna use a sphereBufferGeometry. The args we pass here correspond
to the args for Three.js's SphereBufferGeometry class (Show docs).
Radius, width segments, height segments, etc. We only care about these
first three. If we pass a small number of segments, we'll get something
real chunky. The higher we go, the smoother, but also the more intensive
it will be to render it. Let's try 32, which is kind of a lot but it'll
make it look nice. We can always lower it later if it is laggy.

Next is the material. We'll use a meshStandardMaterial, and set the
color to Colors.shape (this is defined in the utils file). Notice how it
looks black on the screen. That is because this is a lit material. Its
not a bug, the sphere is black because there is no light shining on it.

### Add Lights

Let's fix that by adding some lights to the scene. First, add this
Lights component to the scene. Now, open up Lights.js so we can
implement it.

I already made this DirectionalLight component because I'm too lazy to
type all of this live.

So inside the Lights component, we'll start with a React Fragment. In
there, let's do an ambient light. This is like the base level of
lightness of the scene. We don't want it to be too bright, because we
want some shadows.

Next up, throw our Directional Light in there. You can think of this as
sun light. It has a direction it shines in, but the light rays basically
travel in a straight line.

This looks decent, but we're going to add a couple more. Adding multiple
light sources can create some depth to the scene.
Point lights are kinda like little lightbulbs in the scene. They have a
position and emit light in a sphere around that point. We'll do a
regular white one, and maybe a red one to mix it up.

The difference is subtle, but it'll be more noticeable later.

### Add Floor

So we have our shaded sphere, but it would be nice to have some sort of
floor for the sphere to cast a shadow on. Let's make a Floor component.
We need a mesh. We need to rotate it around the x axis 90 degrees.
Three.js uses radians instead of degrees, so instead of 90 here I am
putting Math.PI / 2. Don't worry if you don't know or remember
trigonometry. Just do what I do and copy it from stack overflow. We'll
also set the y position so its under the sphere. And set this
receiveShadow flag.

The planeBufferGeometry's args are width and height. We want it to be
pretty big.

The shadowMaterial is just a material that can have shadows cast on it.
Easy peasy. Now go ahead and add that to the scene... but we don't see
anything. What gives? Oh, we need to go back to our sphere and give it
the castShadow flag. Maybe let's do the receiveShadow flag too just in
case. BOOM. That is very exciting.

### Fog and Soft shadows

Two more small things.

One: Let's add some fog. We can really make this whatever color we want.
It'll give a slight tint to our scene, which adds depth.

Two: Call this softShadows function from drei. This is a helper to make
really nice, fuzzy shadows.

So that's it for the first section. We're going to take a 15 minute
break. You're going to go into breakout rooms where you can play around
with what we've got so far. Some ideas:
- Tweak colors and positions.
- Change the sphere to a different shape. Consult the three.js docs.
  torusBufferGeometry, boxBufferGeometry, etc.
- Change the amount and properties of the lights.

There should be one or two people in your room that can help out if you
have any questions or get stuck. I'll also post a link to a new codepen
with what I have so far if you want a fresh start or if you are joining
late.

## Step 2

### Factor out shape

For this section, we are going to make our scene more dynamic.

First let's cut out our shape from the scene, then go into Shape.js,
and paste it in.

Now in ShapeCluster.js, lets render a Shape inside of a group component.
A group is kind of like a genera purpose wrapper. We can apply
transforms to it like position, rotation, etc, and it will apply to
everything in the group.

Now, real quick, lets put ShapeCluster in our scene so we can see our
sphere again. Great, but lets get a few more spheres going.

### Random Cluster

Go into Shape, and give it a position prop.

Let's have ShapeCluster take a number prop, for how many shapes to
render. We can give it a default of 10. So we want to make 10 shapes,
and give each one a random position. A good way to calculate random
positions is with React's useMemo hook. This way we can get a random
position once for each sphere. when the component first mounts.

I am going to make a custom hook called usePositions. It will take the
number of positions we want to generate. We're going to call useMemo.
Inside here, we make an array as big as the number of positions we want.

Then, map the array. Each element should be an array containing [xyz]
coordinates. So in the map, we return an array. Now we just need to
generate some random values for x, y, and z.

There are a lot of different ways we could determine these. I'm going
to punch in some random ranges that look good, using this randomRange
helper function.

Now we have our array of positions, we can map each one to a Shape
element. Don't forget to give each one a key. Now we've got this weird
random blob of shapes. It isn't _super_ exciting yet, but we'll get
there. Let's animate the spheres to make them stretch and squish.

### Basic animation

Animation in react-three-fiber is interesting. I told you we'd be
writing declarative code, and that is mostly true. However, for stuff
like animation where we want to update every frame (16ms or so), we
actually want to circumvent React and handle it imperatively. Our scene
would slow to a crawl if we were rerendering a lot of React components
that often.

In the shape, we need a ref to the mesh. Then, we'll use a
react-three-fiber hook called useFrame. We can pass it a function that
will run every frame. The function will get passed a state object that
has a lot of stuff on it, but most importantly it has the time elapsed
since we started the scene.

Animation is just changing a property over time. If I just say
"meshref.current.position.y = time", we will see all of our shapes
slowly rise up. Nice, but we can do better.

### Up-and-down motion 

Let's have the shapes go up and down so they don't just float off
screen. We can achieve this with Math.sin. [SHOW GRAPHIC] If you're not
familiar with trigonometry, this is a mathematical function that repeats
itself every so often, so it gives us that nice back-and-forth motion.

### Varying animation speed 

Now our shapes are going up and down, but it'd be nice if they weren't
in perfect unison. Another good opportunity for a random factor. Let's
make a random offset for each shape. If we multiply it and time, we'll
see our shapes moving at different rates. NICE.

### Easing animation 

We can make the up-down movement more dynamic by applying an _easing_
function. Depending on what easing function we use, we can make our
motion more smooth, more bouncy, more jerky. I've prewritten
easeInOutCubic which we can apply to our t variable.

### Scale anim and metalness

In addition to up and down motion, we can also update the y-scale to
give it a stretchy effect.

While we are here, let's give our shapes a little metallic shine. With
the standard material we can specify some physical properties. You can
refer to the Three.js docs to see what they all are.

### Animate group

So we've animated our shapes, but wouldn't it be cool if we could
animate the entire shape cluster? Let's try it out. Same as before, make
a ref, put it on the _group_, and then call the useFrame hook.

I'm thinking we'll want some back-and-forth motion again, so we'll use
Math.sin. Let's try rotating the entire group around the y axis.

Look at that! That's a pretty nice swivel we've got going on. Let's call
that the end of section two. Now, another break. During the break, if
you'd like, play around with the animations we made. Try adjusting some
of the coefficients, or using different easing functions. If you're
feeling adventurous, try animating different properties, like the scale
of the group, or rotate the spheres on the z-axis, etc.

## Step 3

### Orbit Controls

Now, I could stare at this for hours. But I would really love it if I
could interact with it in some way. Give me something to click on.

First, an easy one. Let's import OrbitControls from drei and toss that
in the bottom of our scene. Now we can zoom with the mouse wheel, rotate
with left click, and pan with right-click (or command click). Now our
scene really feels like a full 3D space we can explore.

### On Click Shape

Next, lets make each shape respond to click events. I'm thinking I want
the shapes to change colors and size when clicked.

Inside the shape component, we'll use the useState hook to keep track of
whether or not our shape has been clicked. Let's add an onClick prop to
the mesh to toggle active

### First useSpring Attempt

Then, we can import and use the useSpring hook from react-spring. This
is an amazing little animation library that works with the dom as well
as r3f (NOTE: Make sure the import is react-spring/three. This version
of the library is specific to r3f).

The object we pass to useSpring can have any properties we want on it.
We're going to give it a color, and make it change depending on the
active state. the hook returns a similar object, with special animated
properties. useSpring is smart enough to tween between lots of different
variable types, even these css colors.

We'll also pass it a config key. We imported these preset configs from
react-spring. I'm going to use config.wobbly because it looks fun, but
you can find others on the react-spring docs.

Now just use that color variable in the material.

### useSpring scale

The color variable returned by useSpring is a special animated value,
and we have to use special animated components that understand them.
This is no big deal, just import animated from react-spring, and put it
infront of your component names like so.

Let's also adjust the scale in the spring. Don't forget to make the
component animated.

And thats it for interactivity. This is really just a basic example of
what you could do. Instead of onClick, you could do hover effects with
onMouseOver/onMouseOut, you could spawn _new_ shapes on click, you could
make the entire group react to your input. The possibilities are really
endless. We are going to do one more break, where you can play around
with react-spring, and then when we come back, maybe we can look at some
peoples' scenes and wrap up.